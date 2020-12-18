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
  -P 
```
