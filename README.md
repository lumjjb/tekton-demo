# A simple tekton example with security/compliance components

This repository contains an example of a pipeline that includes security centric
pipelines. Below are the steps to try out the pipeline.


## Set up a cluster using minikube

For a simple setup, we can use minikube. The version tested on is minikube v1.5.0

```
➜  ~ minikube start --vm-driver=virtualbox
😄  minikube v1.5.0 on Darwin 10.14.6
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.16.2 on Docker 18.09.9 ...
🚜  Pulling images ...
🚀  Launching Kubernetes ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```

## Set up tekton

After the kubernetes cluster is setup, install tekton with the following command:

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

This should install all the necessary comoponents of tekton to get started.

## Set up registry and generate keys for encryption


### Registry setup
For this example, we will use a registry local to the cluster. This can be done by using the provided deployment:
```
kubectl create -f yamls/registry.yaml
```

### Image encryption setup
We should also create a RSA key pair for encryption demonstration with the following commands:
```
openssl genrsa -out private.key 1024
openssl rsa -in private.key -pubout > public.key
```

We can then create a kubernetes secret for encryption key using the following command:

```
kubectl create secret generic enc-key --from-file public.key
```


### Trivy setup
Because we are using trivy as well, we want to update the vulnerability database to be used in the task. We create a daemonset that will update the trivy vulnerability database on the local nodes in the minikube cluster:

```
kubectl create -f yamls/update-trivy.yaml
```

Next watch the logs of the daemonset pods until you see "updated db". You may leave this pod running to update the local cache of the minikube node. 
NOTE: For other setups, some modification to where the cache should be stored.
```
➜  kubectl logs -l app=trivy-update
updated db
```

## Setup task
With that in place, we will define the actual task that we have created, this resource contains the description of all the steps that will be done.

```
kubectl create -f yamls/build-img-task.yaml
```

## Running the task

With all that setup, we can create the resources which will be the input and output of our defined task/pipeline:

```
kubectl create -f yamls/git-pipeline-rsrc.yaml
kubectl create -f yamls/image-pipeline-rsrc.yaml
```

We can then create the task with the following command:
```
kubectl create -f yamls/build-img-taskrun.yaml
```

We can then use the following command to watch the status of the task:

```
watch "tkn taskrun describe build-docker-image-from-git-source-task-run"
```

In addition, logs of each step can be viewed with the following command:

```
tkn taskrun logs -f  build-docker-image-from-git-source-task-run
```
