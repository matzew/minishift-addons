# Deploying Knative on OpenShift

This guide walks you through the installation of the latest version of [Knative
Serving](https://github.com/knative/serving) on an
[OpenShift](https://github.com/openshift/origin) using pre-built images and
demonstrates creating and deploying an image of a sample "hello world" app onto
the newly created Knative cluster.

You can find [guides for other platforms here](README.md).

## Before you begin

These instructions will run an OpenShift 3.10 (Kubernetes 1.10) cluster on your
local machine using [minishift](https://docs.okd.io/latest/minishift/getting-started/index.html)
to test-drive Knative.

## Configure and start minishift

The following details the bare minimum configuration required to setup minishift for running Knative:

```shell
minishift profile set knative
minishift config set memory 8GB
minishift config set cpus 4
minishift config set image-caching true
minishift addon enable admin-user
minishift addon enable anyuid
```

The above configuration ensures that Knative gets created in its own [minishift profile](https://docs.okd.io/latest/minishift/using/profiles.html) with 8GB of RAM and 4 vCpus. The image-caching helps in re-starting up the cluster faster every time.  The [addon](https://docs.okd.io/latest/minishift/using/addons.html) **admin-user** creates a default `admin` user with the role  cluster-admin and the  [addon](https://docs.okd.io/latest/minishift/using/addons.html) **anyuid** allows the `default` service account to run the application with uid `0`.

To start minishift:

```shell
minishift profile set knative
minishift start
```

The command `minishift profile set knative` is required every time you start and stop minishift to make sure that you are on right `knative` minishift profile that was configured above.

## Configuring `oc` (openshift cli)

Running the following command make sure that you have right version of `oc` and have configured the DOCKER daemon to be connected to minishift Docker.

```shell
eval $(minishift docker-env) && eval $(minishift oc-env)
```

## Preparing Knative Deployment

### Enable Admission Controller Webhook
To be able to deploy and run serverless Knative applications, its required that you must enable the [Admission Controller Webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/).  

Run the following command to make OpenShift (run via minishift) to be configured for [Admission Controller Webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/):

```shell
minishift openshift config set --target=kube --patch '{
    "admissionConfig": {
        "pluginConfig": {
            "ValidatingAdmissionWebhook": {
                "configuration": {
                    "apiVersion": "v1",
                    "kind": "DefaultAdmissionConfig",
                    "disable": false
                }
            },
            "MutatingAdmissionWebhook": {
                "configuration": {
                    "apiVersion": "v1",
                    "kind": "DefaultAdmissionConfig",
                    "disable": false
                }
            }
        }
    }
}'
```

Please allow few minutes for the OpenShift to be restarted.

### Download Knative dependencies

Knative comes with its own Istio configuration, and for Openshift we need to download it as well as the Knative installation yaml and apply a few changes.

```
curl -L https://storage.googleapis.com/knative-releases/serving/latest/istio.yaml |  sed 's/LoadBalancer/NodePort/' > istio.yaml 
curl -L https://github.com/knative/serving/releases/download/v0.1.1/release.yaml |  sed 's/LoadBalancer/NodePort/' > knative.yaml  
```

Now, export the folder containing these two files as `KNATIVE_HOME`:

```
KNATIVE_HOME=$(pwd)   
```

## Install add-on

Run the following command to install `knative` add-on:

```
$ minishift addon install <path_to_directory_containing_this_readme>
```

## Deploy Knative

Run the following command to apply `knative` add-on:

```
$ minishift addon apply knative --addon-env KNATIVE_HOME=$KNATIVE_HOME
```

