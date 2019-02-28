# Template Cheat Sheet

# Entire ConfigMap as Environment Variables

```
    spec:
      containers:
        - envFrom:
            - configMapRef:
                name: configmap-name
```

# Run a Pod as a Specific Service Account

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