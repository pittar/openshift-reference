# CLI Cheat Sheet

## New Project with NodeSelector

```
oc adm new-project myproject \
    --node-selector='label.key=label.value'
```

## Serivce Accounts

Create service accounts and apply roles or scc:
```
$ oc create sa cicd
$ oc adm policy add-scc-to-user anyuid -z cicd # Bad, Don't do this!
$ oc adm policy add-role-to-user edit -z cicd
```

## Adding SCCs or Roles to a Service Account

```
# Only admins can add SCCs.
$ oc adm policy add-scc-to-user anyuid -z cicd # Bad, Don't do this unless it's absolutely necessary.
# Normal users can add roles.ß
$ oc adm policy add-role-to-user edit -z cicd
```

## Disable Automatic Deployment

```
oc get dc <dc name> -o yaml -n <project> | sed 's/automatic: true/automatic: false/g' | oc replace -f -
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
