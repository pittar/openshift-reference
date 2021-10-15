# Helpful Scripts

## Resource Export

Export a "clean" reasource that doesn't have all of the fields specific to that particular instance.

**`oc-export`**

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
oc-export service myapp > myapp-service.yaml
```

Or... if `oc-export` is on your path, it will be considered an `oc plugin`, so you can use it as if it were a native `oc` command!

```
oc export service myapp > myapp-service.yaml
```

## New Project with "Image Puller" on a Different Project

One common pattern is to keep the container images that you build in a well known project/namespace - for example "cicd-tools".  By default, a Deployment in one project can't pull an image from a different project.  This is due to the default Role Based Access Control that blocks cross-project resource sharing.

In order for Deployments in the project "andrew-test" to pull images from "cicd-tool", you need to give the `ServiceAccount` used by the `Deployment / DeploymentConfig / Job / etc...` the `system:image-puller` role on the "cicd-tools" project.  If you don't specify a `ServiceAccount` to use, then OpenShift will use the "default" `ServiceAccount` for the project.

A fully qualified `ServiceAccount` name follows the pattern:
`system:serviceaccount:<project name>:<service account name>`

So, the "default" service account in the "andrew-test" project would be:
`system:serviceaccount:andrew-test:default`

To grant this `ServiceAccount` the ability to pull images from "cicd-tools", you would run the following command:
`oc policy add-role-to-user system:image-puller system:serviceaccount:andrew-test:default`

If all new projects should have the ability to pull images from "cicd-tools", then you might want to make a small script that can create the new project and assign this role all in one shot.  For example:

* **new-project.sh** *
```
#!/bin/bash

# Create the new project.
oc new-project $1

# Assign the default service account image puller on cicd-tools.
oc policy add-role-to-user system:image-puller system:serviceaccount:$1:default
```

Then, simply make the file executable and run it like so:

```
# Make this script executable.
chmod a+x new-project.sh

# Create a new project.  It will also be able to pull images from cicd-tools.
./new-project andrew-demo
```
