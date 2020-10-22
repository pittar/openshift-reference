# Minishift Cheat Sheet

**Minishift is for OpenShift 3.11.  For OpenShift 4.x, [please use CodeReady Containers](https://developers.redhat.com/products/codeready-containers/overview)

## Minishift Docs

The latest Minishift documentation [can be found here](https://docs.okd.io/latest/minishift/).

## Install

### MacOS

Make sure you follow the instructions to setup **xhyve**.  This way you won't need VirtualBox.
* [xhyve install instructions](https://docs.okd.io/latest/minishift/getting-started/setting-up-virtualization-environment.html#for-macos)

Once xhyve is ready, the easiset way to install Minishift is with [Homebrew](https://brew.sh/)
* [Install Minishift with Homebrew](https://docs.okd.io/latest/minishift/getting-started/installing.html#installing-with-homebrew)

## Configure

Depending on your system spec, you'll probably want to allocate 1/2 to 3/4 of your resource to the minishift vm before you start it the first time.  To do this:

```
$ minishift config set cpus=6
$ minishift config set memory=12288
$ minishift config set disk-size=100GB
```

I find the following addons helpful as well:
```
$ minishift addons enable admin-user
$ minishift addons enable registry-route
```

I do **not** suggest adding `anyuid`.  If you really need to run a pod with anyuid, create a [service account](cli.md) with `anyuid` and run your pod with that service account.

## Adding Java s2i

For some reason, Java s2i is not present in Minishift.  Here's how to add it.

First, pull the s2i image.  As an admin:

```
$ oc project openshift
$ oc import-image redhat-openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm
$ oc tag redhat-openjdk18-openshift:latest redhat-openjdk18-openshift:1.4
$ oc apply -f templates/java-s2i.yaml
```
