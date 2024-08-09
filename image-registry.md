# Image Registry Configuration

## Block Storage

When RWX and Object storage aren't available, you can configure the image registry to [use Block storage instead]().

First, you need to create a PVC:

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: image-registry-storage 
  namespace: openshift-image-registry 
spec:
  accessModes:
  - ReadWriteOnce 
  resources:
    requests:
      storage: 100Gi 
```

This PVC will use the default storage class in your cluster (usually Block storage).  If your cluster doesn't have to have a default storage class, you'll have to specify it in the PVC.

Next, make sure your image registry is set to a single replica, since Block storage (RWO) can only be mounted to a single pod (or at least, all pods have to be on the same node... but the registry will default to run on multiple nodes).

```
oc patch config.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"rolloutStrategy":"Recreate","replicas":1}}'
```

Finally, you need to edit the registry config to use the new PVC:

```
oc edit config.imageregistry.operator.openshift.io -o yaml
```

Then, set the storage:

```
storage:
  pvc:
    claim: 
```

Provided you used the name `image-registry-storage` for the PVC, you can leave the `claim:` field empty like in the example, it will automatically get populated after you save the config.

Once you save the config, your registry deployment should cycle the pod and you're up and running with a block storage backed volume.