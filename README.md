# OpenShift Reference Card

Some OpenShift commands and template tricks I tend to use, then forget...

## CLI

Create service accounts and apply roles or scc:
```
$ oc create sa cicd
$ oc adm policy add-scc-to-user anyuid -z cicd # Bad, Don't do this!
$ oc adm policy add-role-to-user edit -z cicd
```

## Templates

Consume an entire `ConfigMap` as environment variables:

```
    spec:
      containers:
        - envFrom:
            - configMapRef:
                name: configmap-name
```

Run a container as a specific `service account`:
```
kind: DeploymentConfig
metadata:
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
      ...
      serviceAccountName: specialaccount
```


