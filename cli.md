# CLI Cheat Sheet

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
