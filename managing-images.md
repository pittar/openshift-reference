# Managing Images

[Managing Images Docs](https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html#allowing-pods-to-reference-images-from-other-secured-registries)

## Pulling from External Secure Repositories

In order to pull from an external repository that requires login, you need to create a *secret* with your credentials and link it to your `default` account.

```
$ oc create secret docker-registry <pull_secret_name> \
    --docker-server=<registry_server> \
    --docker-username=<user_name> \
    --docker-password=<password> \
    --docker-email=<email>

$ oc secrets link default <pull_secret_name> --for=pull
```
