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

* From the left navigation, click `Clients`