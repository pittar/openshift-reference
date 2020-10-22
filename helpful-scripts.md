# Helpful Scripts

## Resource Export

Export a "clean" reasource that doesn't have all of the fields specific to that particular instance.

`**oc-export**`

```
#!/bin/bash

oc get $1 $2 -o yaml \
    | yq d - 'metadata.resourceVersion' \
    | yq d - 'metadata.uid' \
    | yq d - 'metadata.annotations' \
    | yq d - 'metadata.creationTimestamp' \
    | yq d - 'metadata.selfLink' \
    | yq d - 'metadata.managedFields' \
    | yq d - 'metadata.generation' \
    | yq d - 'spec.template.metadata.creationTimestamp' \
    | yq d - 'status'
```

Usage:
```
oc export service myapp > myapp-service.yaml
```
