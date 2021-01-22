# Installing Red Hat SSO and Configuring OpenShift OAuth

Red Hat Single Sign-On (RH-SSO) is based on the *upstream* [keycloak](https://www.keycloak.org/) and provides a powerful and flexible identity management solution.
Using RH-SSO to manage OpenShift Authentication can be a handy way to manage users and federate identity from your corporate LDAP, Active Directory, or any number of 
identity providers that offer OpenID Connect or SAML2 interfaces.

## Install Red Hat SSO 7.4 from the Template

As of RH-SSO 7.4, the best installation method is with the template.  There is a RH-SSO Operator that is currently in alpha, but it isn't production ready yet.

### Create a Project/Namespace

First, create a new project for Red Hat SSO.  You can use whatever you like, but I'll use `rh-sso` in these docs.

```
$ oc new-project rh-sso
```

### Install Red Hat SSO from the Template

Next, install Red Hat SSO using the template.  You can do this through the GUI, or the command line like so:

```
$ oc new-app --template=sso74-ocp4-x509-postgresql-persistent \
  -p SSO_ADMIN_USERNAME=admin \
  -p SSO_ADMIN_PASSWORD=password
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
