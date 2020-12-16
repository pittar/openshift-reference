# Mounting ConfigMaps and Secrets as Volumes

Sometimes it can be handy to add files (e.g. `application.properties`) to a `ConfigMap` or `Secret` and mount them in a well-known location in your container file system.

## Create a ConfigMap or Secret From a File

First, let's assume:
1. You are logged into an OpenShift cluster with the `oc` cli.
2. You are in a directory that contains two files you want to include in your `ConfigMap`: "application.properties" and "values.txt"

They syntax for creating the `ConfigMap` is:

```
oc create configmap <configmap name> \
  --from-file=<filename1> \
  --from-file=<filename1> \
  --from-file=<filename1> # ... etc, etc, etc...
```

If your files exist in a different directory, or you want to use a different name for the file in your `ConfigMap`, you can use:

```
oc create configmap <configmap name> \
  --from-file=<filename1 in configmap>=<path/to/file1> \
  --from-file=<filename2 in configmap>=<path/to/file2> \
  --from-file=<filename3 in configmap>=<path/to/file3> # ... etc, etc, etc...
```

You will now have a `ConfigMap` in your project with an entry for each file, where the "filename" is the key, and the contents are the value.

Back to the original example of two files in the same directory that we are running `oc` from called "application.properties" and "values.txt", the command would be:

```
oc create configmap custom-config \
  --from-file=application.properties \
  --from-file=values.txt
```

## Mounting ConfigMap as a Volume

Now that you have your `ConfigMap`, it's time to mount it as a volume.

In your `Deployment` of `DeploymentConfig`, add a new `volume` to your pod template spec.  This example has an `emptyDir` volume for data (who wants to keep their data anyway?) and another volume that references the `ConfigMap`:

```
    spec:
      volumes:
        - name: data-volume
          emptyDir: {}
        - name: config-volume
          configMap:
            name: custom-config
            defaultMode: 420
```

Then, in the container stanza of the pod spec, you mount the volume to a well known location:

```
          volumeMounts:
            - name: data-volume
              mountPath: /var/lib/pgsql/data
            - name: config-volume
              mountPath: /opt/app-root/src/custom-config
```

Once your pod restarts, you will your files in the specified directory:

```
$ ls -l /opt/app-root/src/custom-config
application.properties
values.txt
```

A more fleshed out example of a `DeploymentConfig` (with some boiler place yaml deleted for brevity) with a mounted ConfigMap:

```
kind: DeploymentConfig
apiVersion: apps.openshift.io/v1
metadata:
  name: custompg
spec:
  selector:
    name: custompg
  template:
    metadata:
      labels:
        name: custompg
    spec:
      containers:
        - name: postgresql
          ports:
            - containerPort: 5432
              protocol: TCP
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: custompg-data
              mountPath: /var/lib/pgsql/data
            - name: config-volume
              mountPath: /opt/app-root/src/postgresql-start
          terminationMessagePolicy: File
      volumes:
        - name: custompg-data
          emptyDir: {}
        - name: config-volume
          configMap:
            name: pg-config
            defaultMode: 420
      restartPolicy: Always
      dnsPolicy: ClusterFirst
```
