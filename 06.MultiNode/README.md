# Kubernetes Multi Node Cluster Setup Step By Step | Kind

Check the kind [website](https://kind.sigs.k8s.io/docs/user/quick-start/) for steps to install kind on various operating systems. 

install kind on MacOS using brew.
```
brew install kind
```
![alt text](<images/Screenshot 2024-10-31 at 13.53.30.png>)

I have Kubectl already installed on my system. If you don't have follow this [link](https://kubernetes.io/docs/tasks/tools/) to install it. We use it to interract with the kind cluster

Below domains and all of its sub-domains are allowed to be referred in the exam...

- https://kubernetes.io/docs
- https://kubernetes.io/blog/
- Kubernetes cheat sheet : https://kubernetes.io/docs/reference/kubectl/quick-reference/


## Creating a cluster

Check for the [release note](https://github.com/kubernetes-sigs/kind/releases) to select the version you want to install. If no version is selected, it will install the latest version. I will be using verion v1.30.4

kind create cluster --image  kindest/node:v1.30.4@sha256:976ea815844d5fa93be213437e3ff5754cd599b040946b5cca43ca45c2047114 --name cka-cluster1

![alt text](<images/Screenshot 2024-10-31 at 14.10.36.png>)

If we are working on multiple cluster we need to run this command to check on the cluster we are working on
```
kubectl cluster-info --context kind-cka-cluster1
```
![alt text](<images/Screenshot 2024-10-31 at 14.14.54.png>)

Check the nodes using kubectl
```
kubectl get nodes
```
![alt text](<images/Screenshot 2024-10-31 at 14.16.30.png>)

## Creating Multi-node clusters

In particular, many users may be interested in multi-node clusters. A simple configuration for this can be achieved with the following config file contents:

Create a Yaml file and paste the following configuration inside
```
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```
![alt text](<images/Screenshot 2024-10-31 at 14.20.40.png>)

The configuration above contains a single control plane and two worker nodes. Create the cluster with following command.
```
kind create cluster --image  kindest/node:v1.30.4@sha256:976ea815844d5fa93be213437e3ff5754cd599b040946b5cca43ca45c2047114 --name cka-cluster2 --config config.yaml
```
![alt text](<images/Screenshot 2024-10-31 at 15.52.21.png>)

Get the nodes
```
Kubectl get nodes
```
![alt text](<images/Screenshot 2024-10-31 at 15.53.27.png>)

Because we just created a new cluster, that is where it is fetching its context from. To select a particular cluster, we need to set the context. 

For kubernetes command cheat code, go to this [site](https://kubernetes.io/docs/reference/kubectl/quick-reference/). Search for cheats in the search bar

To check the list of cluster you have present, use this command
```
kubectl config get-contexts
```
To delete any context
```
kubectl config delete-context NAME

```

To set context of the first cluster, use this command;
```
kubectl config set-context --current --namespace=kind-cka-cluster1
```

to switch to the first clustet use this command
```
kubectl config use-context kind-cka-cluster1
```
![alt text](<images/Screenshot 2024-10-31 at 17.28.21.png>)
