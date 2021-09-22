# Installing Red Hat SSO and Configuring OpenShift OAuth

Red Hat Single Sign-On (RH-SSO) is based on the *upstream* [keycloak](https://www.keycloak.org/) and provides a powerful and flexible identity management solution.
Using RH-SSO to manage OpenShift Authentication can be a handy way to manage users and federate identity from your corporate LDAP, Active Directory, or any number of 
identity providers that offer OpenID Connect or SAML2 interfaces.

## Prerequsite: Confirm Templates Are Up To Date

Login to your OpenShift cluster with the `oc` command line tool as a *Cluster Admin*.

Run the following command to see if you already have the Red Hat Single Sign-On 7.5 templates in your cluster.

```
$ oc get template -n openshift | grep sso75
```

If not, then run the following commands to import them.

```
$ for resource in sso75-image-stream.json \
  sso75-https.json \
  sso75-postgresql.json \
  sso75-postgresql-persistent.json \
  sso75-x509-https.json \
  sso75-x509-postgresql-persistent.json
do
  oc replace -n openshift --force -f \
  https://raw.githubusercontent.com/jboss-container-images/redhat-sso-7-openshift-image/sso75-cpaas-dev/templates/${resource}
done
```

Ignore any warnings along the lines of `Using non-groupfied API resources is deprecated and will be removed in a future release, update apiVersion to "template.openshift.io/v1" for your resource`.

Finally, run the following command to import the Red Hat Single Sign-On 7.5 *ImageStream*.

```
$ oc -n openshift import-image rh-sso-7/sso75-openshift-rhel8:7.5 --from=registry.redhat.io/rh-sso-7/sso75-openshift-rhel8:7.5 --confirm
```

## Install Red Hat SSO 7.5 from the Template

As of Red Hat Single Sign-On 7.5, the best installation method is with the template.  There is an Operator that is currently in *Tech Preview*, but it isn't production ready yet.

### Create a Project/Namespace

First, create a new project for Red Hat Single Sign-On.  You can use whatever you like, but I'll use `rh-sso` in these docs.

```
$ oc new-project rh-sso
```

Also, if you plan on using the default PostgreSQL container available in the OpenShift catalog with this deployment, be sure the `10` tag exists.

```
$ oc get is -n openshift | grep postgresql
```

If not (for example, if you have `10-el8` instead), the simplest solution is to create the tag:

```
$ oc tag postgresql:10-el8 postgresql:10 -n openshift
```

### Install Red Hat SSO from the Template

Next, install Red Hat SSO using the template.  You can do this through the GUI, or the command line like so:

```
$ oc new-app --template=sso75-x509-postgresql-persistent \
  -p SSO_ADMIN_USERNAME=admin \
  -p SSO_ADMIN_PASSWORD=password
  
# Once RH-SSO is fully started, remove the env vars for the admin user/pass.  This is so the 
# admin user/pass doesn't get re-created on every pod start.  The original settings will be
# persisted in the PostgreSQL database.

oc set env dc/sso SSO_ADMIN_USERNAME-
oc set env dc/sso SSO_ADMIN_PASSWORD-
```

### Create an OpenShift Realm and Client

The name of the Realm and Client can be whatever you like, as long as you are consistent in their use for the rest of the steps.

#### 1. Login to Red Hat SSO

Login to your new Red Hat SSO instance with the default username/password that you passed into the template.  It's a good idea to change this as soon as you login for the firs time.

#### 2. Create A New Realm

From the Admin console, create a new **Realm**:
* Hover over Realm drop-down list at the top-right of the screen.  It will say `Master` and have a down arrow beside it.
* Click the blue `Add realm` button that appears.
* Enter your realm name (e.g. `openshift`).  Leave all defaults in place and click **Create**.

#### 3. Create A New Client

You should now be in your new Realm (the name will be reflected at the top of the left navigation panel).

* From the left navigation, click `Clients`.
* From the top-right of the client table, click the `Create` button.
* Enter your new *Client ID* (for example, `openshift`).  Stick with the defaults and click `Save`.
* You will now be in the `Settings` tab of your new Client.
* Change `Access Type` to **confidential**.
* Set the `Valid Redirect URIs` to `https://oauth-openshift.apps.<cluster url>/oauth2callback/rhsso`
* **Save**
* Click on the `Credentials` tab at the top of the screen that has now appeared.
* Copy the `Secret`.  You will need it in the next step.

#### 4. Create a Client Secret

Create a Secret to hold the new client secret.

For example: `client-secret.yaml`
```
apiVersion: v1
kind: Secret
metadata:
  name: openid-client-secret
  namespace: openshift-config
type: Opaque
stringData:
  clientSecret: <client secret>
```

Then apply it:
```
$ oc create -f client-secret.yaml -n openshift-config
```

#### 4. Create Ingress Secret

In order for the OAuth server to trust ingress, we need to make a copy of the Ingress certificate and create it as a ConfigMap in the `openshift-config` namespace.

```
# Get name of certs secret.  It can be router-certs or router-certs-default.
CERT_SECRET=$(oc get secrets -n openshift-ingress | grep 'router-certs\|ingress-certs\|-ingress' | cut -d ' ' -f1)

# Get the certificate.
tlscert=`oc get secrets/$CERT_SECRET -o jsonpath={.data.'tls\.crt'} -n openshift-ingress | base64 --decode`

# Create a ConfigMap in the openshift-config namespace that contains the ingress certificate.
oc create configmap openidcacrt --from-literal ca.crt="$tlscert" -n openshift-config
```

#### 5. Add Red Hat SSO as an OAuth Provider

From the OpenShift Admin console:

* Select `Administration` from the left navigation.
* Select `Cluster Setting`.
* Select the `Global Configuration` tab.
* Select `OAuth` from the list.
* Click on the `YAML` tab.
* Add the following to the `identityProvider:` list:

```
    - name: rhsso
      mappingMethod: claim
      type: OpenID
      openID:
        clientID: openshift
        clientSecret: 
          name: openid-client-secret
        ca: 
          name: openidcacrt
        extraScopes: 
        - email
        - profile
        extraAuthorizeParameters: 
          include_granted_scopes: "true"
        claims:
          preferredUsername: 
          - preferred_username
          - email
          name: 
          - nickname
          - given_name
          - name
          email: 
          - custom_email_claim
          - email
        issuer: https://sso-rh-sso.apps.<cluster url>/auth/realms/openshift
```

For example, if you ONLY have Red Hat SSO, your OAuth resource might look like:

```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
    - name: rhsso
      mappingMethod: claim
      type: OpenID
      openID:
        clientID: openshift
        clientSecret: 
          name: openid-client-secret
        ca: 
          name: openidcacrt
        extraScopes: 
        - email
        - profile
        extraAuthorizeParameters: 
          include_granted_scopes: "true"
        claims:
          preferredUsername: 
          - preferred_username
          - email
          name: 
          - nickname
          - given_name
          - name
          email: 
          - custom_email_claim
          - email
        issuer: https://sso-rh-sso.apps.<cluster url>/auth/realms/openshift
```
