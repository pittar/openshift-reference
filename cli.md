# CLI Cheat Sheet

## New Project with NodeSelector

```
oc adm new-project myproject \
    --node-selector='label.key=label.value'
```

## Serivce Accounts

### Create service accounts and apply roles:

```
$ oc create sa cicd
$ oc adm policy add-role-to-user edit -z cicd
```

### Get Service Account Token

```
$ oc sa get-token <service account name>
```

## Disable Automatic Deployment

### OpenShift 3.x

```
$ oc get dc <dc name> -o yaml -n <project> | sed 's/automatic: true/automatic: false/g' | oc replace -f -
```

### OpenShift 4.x

```
$ oc patch ...
```

## Get GitHub Webhook Trigger Secret

```
oc get bc/<build name> -o=jsonpath='{.spec.triggers..github.secret}{"\n"}'
```

To get the Webhook URL:

```
oc describe bc/<build name>
```

Then look for `Webhook Github:` and copy the `URL:`.  You will have to replace `<secret>` with the secret you retrieved in the first step.

## Set Environment Variable

You can easily set an env var on a deployment config from the cli:

```
oc set env dc/<dc name> key=value
```

## OpenShift 4 - Expose Image Registry On Custom Route

```
oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"routes":[{"name":"public-routes","hostname":"registry.apps.<cluster url>"}]}}' --type=merge
```

## Trigger Rollout on a Deployment

Triggering a rollout on a `Deployment` is slightly different than a `DeploymentConfig`:

```
oc rollout restart deployment <deployment name> -n <namespace>
```
