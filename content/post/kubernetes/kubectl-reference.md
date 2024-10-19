+++
title = 'Kubectl CLI Reference'
date = 2024-10-19T10:00:02Z
draft = false
description = "Kubectl is a command-line interface for managing Kubernetes clusters."
featured = false
tags = [
    "kubernetes"
]
categories = ["kubernetes"]
thumbnail = "images/kubernetes.png"
+++
# Kubectl CLI Reference

Kubectl is a command-line interface for managing Kubernetes clusters. The kubectl version must be within one minor version difference of the Kubernetes cluster. For example, a v1.2 client should work with v1.1, v1.2, and v1.3 masters.
<!--more-->
## Installation

#### Kubectl can be installed on Ubuntu, Debian, CentOS, and RedHat operating systems.

### Ubuntu / Debian

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

### CentOS / RedHat

#### For more information about kubectl installation methods, refer to the Kubernetes documentation.

```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```

## Shell Completion

#### To easily manage Kubernetes resources via the command line, you can enable shell completion by adding the completion script to your shell profile.

### Bash (Linux/macOS)

#### If running Bash 3.2 (included with macOS):

```
brew install bash-completion
```

#### For Bash 4.1+:

```
brew install bash-completion@2
```

#### If kubectl was installed via Homebrew, bash completion should work immediately. Otherwise, add the following to your bash profile:

```
kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl
```

#### On Linux, load kubectl completion for bash in the current shell:

```
source <(kubectl completion bash)
```

#### To persist this across sessions, add the completion script to your `.bash_profile`:

```
kubectl completion bash > ~/.kube/completion.bash.inc
printf "
# Kubectl shell completion
source '$HOME/.kube/completion.bash.inc'
" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Zsh

#### To load kubectl completion for zsh into the current shell:

```
source <(kubectl completion zsh)
```

## Syntax

Kubectl uses a simple syntax to manage objects in a Kubernetes cluster:

```
kubectl [command] [TYPE] [NAME] [flags]
```

- **command**: Specifies the operation you want to perform (e.g., create, get, describe, delete).
- **TYPE**: The resource type (case-insensitive; supports singular, plural, or abbreviation).
- **NAME**: The resource name (case-sensitive). If omitted, details for all resources are shown.
- **flags**: Optional flags to modify the command's behavior.

## Useful Basic Commands

### Create

#### Create resources from a file or stdin.

```
# Create a pod using the configuration in pod.json
kubectl create -f ./pod.json

# Create all resources in a folder
kubectl create -f <folder_name>
```

### Delete

#### Delete resources by filenames, stdin, resource name, or label selector.

```
# Delete a pod using the configuration in pod.json
kubectl delete -f ./pod.json

# Delete a pod with the specified name
kubectl delete pod <pod_name>

# Delete all pods
kubectl delete pods --all
```

### Edit

#### Edit a resource in the default editor.

```
# Edit the service named 'docker-registry'
kubectl edit svc/docker-registry
```

### Expose

#### Expose a resource as a new Kubernetes service.

```
# Expose a replication controller nginx as a service on port 80
kubectl expose rc nginx --port=80 --target-port=8000
```

### Get

#### Display information about one or more resources.

```
# List all pods
kubectl get pods

# List pods with additional details (such as node name)
kubectl get pods -o wide

# Get a single pod in JSON format
kubectl get pod <pod-name> -o json
```

### Run

#### Create and run a particular image in the cluster.

```
# Run a single instance of nginx
kubectl run nginx --image=nginx

# Run a replicated instance of nginx
kubectl run nginx --image=nginx --replicas=5
```

### Set

#### Configure application resources.

```
# Update deployment 'registry' with a new environment variable
kubectl set env deployment/registry STORAGE_DIR=/local

# Set a deployment's nginx container image to 'nginx:1.9.1'
kubectl set image deployment/nginx nginx=nginx:1.9.1
```

### Autoscale

#### Automatically scale the number of pods in a deployment based on resource usage.

```
kubectl autoscale deployment foo --min=2 --max=10
```

### Rollout

#### Manage the rollout of a deployment.

```
# Rollback to the previous deployment
kubectl rollout undo deployment/abc
```

### Scale

#### Set a new size for a Deployment, ReplicaSet, or StatefulSet.

```
# Scale a deployment named 'nginx' to 3 replicas
kubectl scale --replicas=3 deployment/nginx
```

## Useful Cluster Management Commands

### Cluster Info

#### Display the addresses of the master and Kubernetes services.

```
kubectl cluster-info
```

### Cordon / Uncordon

#### Mark a node as schedulable or unschedulable.

```
# Mark node "foo" as unschedulable
kubectl cordon foo

# Mark node "foo" as schedulable
kubectl uncordon foo
```

### Drain

#### Prepare a node for maintenance by evicting all its pods.

### Taint

#### Apply taints to a node.
#### Update the taints on one or more nodes.

```
# Drain node "foo", even if there are pods not managed by a ReplicationController, R
$ kubectl drain foo --force

# As above, but abort if there are pods not managed by a ReplicationController, Repl
$ kubectl drain foo --grace-period = 90

#Drain node by ignoring Deamonsets
kubectl drain <node_name> --ignore-daemonsets
```

### Top

#### Display Resource (CPU/Memory/Storage) usage.

```
# Show metrics for all nodes
kubectl top node

# Show metrics for a given node
kubectl top node NODE_NAME

# Show metrics for all pods in the default namespace
kubectl top pod

# Show metrics for all pods in the given namespace
kubectl top pod --namespace = NAMESPACE

# Show metrics for a given pod and its containers
kubectl top pod POD_NAME --containers

# Show metrics for the pods defined by label name=myLabel
kubectl top pod -l name = myLabel
```
## Useful troubleshooting and debugging commands

### Describe

#### Show details of a specic resource or group of resources.

```
# Update node 'foo' with a taint with key 'dedicated' and value 'special-user' and e
# If a taint with that key and effect already exists, its value is replaced as speci
kubectl taint nodes foo dedicated = special-user:NoSchedule

# Remove from node 'foo' the taint with key 'dedicated' and effect 'NoSchedule' if o
kubectl taint nodes foo dedicated:NoSchedule-

# Remove from node 'foo' all the taints with key 'dedicated'
kubectl taint nodes foo dedicated-

# Add a taint with key 'dedicated' on nodes having label mylabel=X
kubectl taint node -l myLabel = X dedicated = foo:PreferNoSchedule
```

### Exec

#### Execute a command in a container.

### Logs

#### Print the logs for a container in a pod or specied resource. If the pod has only one container, the

#### container name is optional.

```
# Describe a node
kubectl describe nodes kubernetes-node-emt8.c.myproject.internal

# Describe a pod
kubectl describe pods/<pod-name>

# Describe a pod identified by type and name in "pod.json"
kubectl describe -f pod.json

# Describe all pods
kubectl describe pods

# Describe pods by label name=myLabel
kubectl describe po -l name = myLabel

# Describe all pods managed by the 'frontend' replication controller (rc-created pod
# get the name of the rc as a prefix in the pod the name).
kubectl describe pods frontend

# Get output from running 'date' from pod 123456-7890, using the first container by
kubectl exec 123456-7890 date

# Get output from running 'date' in ruby-container from pod 123456-
kubectl exec 123456-7890 -c ruby-container date

# Switch to raw terminal mode, sends stdin to 'bash' in ruby-container from pod 1234
# and sends stdout/stderr from 'bash' back to the client
kubectl exec 123456-7890 -c ruby-container -i -t -- bash -il
```

```
# Return snapshot logs from pod nginx with only one container
kubectl logs nginx

# Return snapshot logs for the pods defined by label app=nginx
kubectl logs -lapp = nginx

# Return snapshot of previous terminated ruby container logs from pod web-
kubectl logs -p -c ruby web-

# Begin streaming the logs of the ruby container in pod web-
kubectl logs -f -c ruby web-

# Display only the most recent 20 lines of output in pod nginx
kubectl logs --tail = 20 nginx

# Show all logs from pod nginx written in the last hour
kubectl logs --since = 1h nginx

# Return snapshot logs from first container of a job named hello
kubectl logs job/hello

# Return snapshot logs from container nginx-1 of a deployment named nginx
kubectl logs deployment/nginx -c nginx-
```
## Proxy

#### Creates a proxy server or application-level gateway between localhost and the Kubernetes API

#### Server. It also allows serving static content over specied HTTP path. All incoming data enters

#### through one port and gets forwarded to the remote kubernetes API Server port, except for the path

#### matching the static content path.

```
# To proxy all of the kubernetes api and nothing else, use:
$ kubectl proxy --api-prefix = /

# To proxy only part of the kubernetes api and also some static files:
$ kubectl proxy --www = /my/files --www-prefix = /static/ --api-prefix = /api/
# The above lets you 'curl localhost:8001/api/v1/pods'.

# To proxy the entire kubernetes api at a different root, use:
$ kubectl proxy --api-prefix = /custom/
# The above lets you 'curl localhost:8001/custom/api/v1/pods'

# Run a proxy to kubernetes apiserver on port 8011, serving static content from ./lo
kubectl proxy --port = 8011 --www = ./local/www/
```

## Useful advanced commands

### Apply

#### Apply a conguration to a resource by lename or stdin. The resource name must be specied. This

#### resource will be created if it doesn’t exist yet. To use ‘apply’, always create the resource initially with

#### either ‘apply’ or ‘create –save-cong’.

## Useful settings commands

### label

#### Update the labels on a resource.

```
# Run a proxy to kubernetes apiserver on an arbitrary local port.
# The chosen port for the server will be output to stdout.
kubectl proxy --port = 0

# Apply the configuration in pod.json to a pod.
kubectl apply -f ./pod.json

# Apply the JSON passed into stdin to a pod.
cat pod.json | kubectl apply -f -

# Note: --prune is still in Alpha
# Apply the configuration in manifest.yaml that matches label app=nginx and delete a
kubectl apply --prune -f manifest.yaml -l app = nginx

# Apply the configuration in manifest.yaml and delete all the other configmaps that
kubectl apply --prune -f manifest.yaml --all --prune-whitelist = core/v1/ConfigMap

# Update pod 'foo' with the label 'unhealthy' and the value 'true'.
kubectl label pods foo unhealthy = true

# Update pod 'foo' with the label 'status' and the value 'unhealthy', overwriting an
kubectl label --overwrite pods foo status = unhealthy

# Update all pods in the namespace
kubectl label pods --all status = unhealthy

# Update a pod identified by the type and name in "pod.json"
```

## Useful other commands

### Config

#### Modify kubeconfig files 
```
kubectl label -f pod.json status = unhealthy

# Update pod 'foo' only if the resource is unchanged from version 1.
kubectl label pods foo status = unhealthy --resource-version = 1

# Update pod 'foo' by removing a label named 'bar' if it exists.
# Does not require the --overwrite flag.
kubectl label pods foo bar-

# Display the current-context
kubectl config current-context

# Delete the minikube cluster
kubectl config delete-cluster minikube

# Delete the context for the minikube cluster
kubectl config delete-context minikube

# List the clusters kubectl knows about
kubectl config get-clusters

# List the context kubectl knows about
kubectl config get-contexts

# Rename the context 'old-name' to 'new-name' in your kubeconfig file
kubectl config rename-context old-name new-name

# Set only the server field on the e2e cluster entry without touching other values.
kubectl config set-cluster e2e --server = https://1.2.3.

# Embed certificate authority data for the e2e cluster entry
kubectl config set-cluster e2e --certificate-authority = ~/.kube/e2e/kubernetes.ca.crt

# Disable cert checking for the dev cluster entry
kubectl config set-cluster e2e --insecure-skip-tls-verify = true

# Set the user field on the gce context entry without touching other values
kubectl config set-context gce --user = cluster-admin
```






