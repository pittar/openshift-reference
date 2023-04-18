# Assigning SCCs As RoleBindings

It's not a good practices to assign a `scc` directly to a user or `serviceaccount`.  For example, this works, but is hard to manage:

```
oc adm policy add-sssc-to-user anyuid -z default
```

A better option is to create a `RoleBinding` to bind a particular `scc`, for example:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-anyuid-scc
  namespace: <namespace>
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:openshift:scc:anyuid
subjects:
- kind: ServiceAccount
  name: <service account name>
  namespace: <serivce account namespace>
```
