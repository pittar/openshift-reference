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
