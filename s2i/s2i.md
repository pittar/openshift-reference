# Source-to-Image (S2I)

## UBI 8 OpenJDK 8 and 11 Builds

OpenShift 4 ships with [Universal Base Image (UBI)](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) OpenJDK 8 and 11 base images.  These can be used for both `source` and `binary` based [source-to-image](https://docs.openshift.com/container-platform/4.6/builds/understanding-image-builds.html#build-strategy-s2i_understanding-image-builds) builds.

## Get UBI8 OpenJDK Images and Tags

You can check to see if your OpenShift cluster has these image present by running:

```
$ oc get is -n openshift
```

This will list all the ImageStreams in the `openshift` namespace.  You can grep for the ones you're interested in:

```
$ oc get is -n openshift | grep ubi8-openjdk
```

This will return the UBI 8 OpenJDK 8 and 11 `ImageStream`s.  If you don't see them, or you are missing the tag you want, you can import them with:

```
$ oc import-image ubi8-openjdk-8:<tag> --from=registry.access.redhat.com/ubi8/openjdk-8:<tag> -n openshift --confirm
```

You can find the [latest versions of these images here](https://catalog.redhat.com/software/containers/search?q=ubi8-openjdk).


### "Binary" Builds

**Binary** builds are ideal as part of a CI/CD pipeline where the majority of the build is handled as part of the CI/CD process and OpenShift is only responsible for building a standard container image from the binary produced by the pipeline.

For example, in a standard Java pipeline, a pipeline might looks something like:

1. Clone git repository
2. Run `mvn deploy` to:
    * build the application binary (e.g. executable `jar`)
    * run unit tests
    * upload artifact to enterprise maven repository
3. Run `mvn sonar:sonar` for produce code quality and vulnerability reports
4. Trigger `Source-to-Image` build to produce a standard container image based on the artifact built in step 2.
    * for example `oc start-build my-app-build --from-file=target/app.jar --follow`
5. Rollout new application image to `dev`
6. ... run integraiton tests, rollout to more environments, etc...

There are a few obvious benefits of this process:
1. Use any CI/CD tool you like!  The pipeline above could be created with Jenkins, Tekton, Azure DevOps, Travis, etc...
2. Your CI/CD server doesn't have to build container images!  It's only responsibility for building the Java artifact and triggering the binary build process.

#### Example S2I Binary Build

Here is an example s2i binary build based on OpenJDK 8:

```
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  labels:
    app: appname
    name: appname-build
  name: appname-s2i-build
spec:
  failedBuildsHistoryLimit: 5
  nodeSelector: {}
  output:
    to:
      kind: ImageStreamTag
      name: appname:latest
  postCommit: {}
  resources: {}
  runPolicy: Serial
  source:
    binary: {}
    type: Binary
  strategy:
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: ubi8-openjdk-8:1.3
        namespace: openshift
    type: Source
  successfulBuildsHistoryLimit: 5
```

They key components are:
* Metadata name: `appname-s2i-build` - the name to use when starting the build.
* Spec output: This build outputs a new image to the `appname` ImageStream in the same project, with the `latest` tag.
* Source type: **Binary** - this is a binary build, not a source build.
* Strategy sourceStrategy: Use the `ubi8-openjdk-8:1.3` builder from the `openshift` nammespace to build the image.

### Triggering Binary Build from a Pipeline

It really doesn't matter what continuous integration tool you use, provided it can execute the `oc` command line tool!

For example, here is a snippet of a [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/) that first builds an executable `jar` file with Maven, then calls the *binary build* from the example above in the `cicd` namespace:

```
    stage('Checkout') {
      steps {
        echo "Checkout source."
        checkout scm
      }
    }
    stage('Build JAR') {
      steps {
        echo "Build the app and upload artifact to Nexus."
        sh "mvn deploy"
      }
    }                 
    stage('Build Image') {
      steps {
        script {
          echo "Build container image."
          openshift.withCluster() {
            openshift.withProject('cicd') {
              sh "oc start-build appname-s2i-build --from-file=target/app.jar -n cicd --follow"
            }
          }
        }
      }
    }
```

This sample pipeline also makes use of the Jenkins OpenShift DSL, but the idea is the same.  They key line is:
```
sh "oc start-build appname-s2i-build --from-file=target/app.jar -n cicd --follow"
```

This starts the `appname-s2i-build` build in the `cicd` namespace and uses the `app.jar` binary from the previous maven stage as the source for the build.

