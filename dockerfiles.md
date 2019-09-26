# Dockerfiles

If your application needs access to certain files or directories, makes sure to add this command in your Dockerfile:

```
RUN chgrp -R 0 /some/directory && \
    chmod -R g=u /some/directory
```

This is based on: [Creating Images Guidelines](https://docs.openshift.com/container-platform/3.11/creating_images/guidelines.html#openshift-specific-guidelines)

As of OpenShift 4.x, there is no need to insert the random UID into `/etc/passwd`, as CRIO does this for you.
