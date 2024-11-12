# Namespaces

In Kubernetes, a **namespace** is a way to divide cluster resources between multiple users or teams, essentially providing a "virtual cluster" within the physical Kubernetes cluster. Namespaces enable you to organize, manage, and isolate resources, making it easier to handle large clusters with many users or projects.

Key Points About Kubernetes Namespaces:
Logical Partitioning: Namespaces logically partition the cluster so that different teams or projects can have isolated environments within the same physical cluster. Each namespace has its own set of resources, such as pods, services, and deployments, separate from other namespaces.

Isolation and Access Control: Namespaces provide isolation for resources, helping prevent conflicts (e.g., two resources with the same name) and allowing control over who can access which resources. This isolation can be enhanced with Role-Based Access Control (RBAC), allowing you to define different access permissions for each namespace.

Resource Quotas: You can assign specific resource limits (e.g., CPU, memory) to namespaces using resource quotas. This allows you to manage and control the amount of resources each team or project can use, preventing any single namespace from consuming excessive cluster resources.

Network Policies: Namespaces can be configured with network policies to restrict communication between resources in different namespaces, further enhancing isolation and security.

## When to Use Namespaces

- Multi-Tenant Clusters: In environments where multiple teams, departments, or applications are sharing the same Kubernetes cluster, namespaces allow each team to have a dedicated workspace.
- Environment Separation: Namespaces can be used to separate environments, such as development, staging, and production, within the same cluster. This helps manage resource allocation and control permissions between environments.

## Default Namespaces in Kubernetes

Kubernetes includes some default namespaces:

- default: This is the namespace for resources that don’t have a specific namespace assigned.
- kube-system: Contains Kubernetes system components, such as the API server and controller manager.
- kube-public: Generally used for resources that should be publicly accessible across all users.
- kube-node-lease: Used by Kubernetes for node heartbeat tracking (helps determine node availability).

```
kubectl get namespaces
```
![alt text](<images/Screenshot 2024-11-11 at 12.50.26.png>)

Any resources we created without signifying any namespace when creating it automticaly will be created under default namespace. So  we expect to see all the resources we created back on the default namesapce.

![alt text](<images/Screenshot 2024-11-11 at 12.54.50.png>)

```
kubectl get all -n kube-system
```

![alt text](<images/Screenshot 2024-11-11 at 13.04.13.png>)

 To create a demo namespace lets run the command below saving the configuration in ns.yaml

 ``
 vi ns.yaml
 ``

 It will automatically open a vi editor

 ![alt text](<images/Screenshot 2024-11-11 at 13.14.30.png>)

 lets apply the documment with

 ```
 k apply -f ns.yaml
 ```
You can also create it imperately with:
```
kubectl create ns demo
```
 ```
 k get ns
 ```
 ![alt text](<images/Screenshot 2024-11-11 at 13.16.34.png>)

 lets create a deployment in our new workspace

 ```
 kubectl create deploy nginx-demo --image=nginx -n demo -o yaml > nginx-demo.yaml
 ```
 To check the resources we created in the demo namespace we have to specify the namespace.
```
k get deploy -n demo
```
![alt text](<images/Screenshot 2024-11-11 at 13.25.21.png>)

We already have nginx-dep on our default ns and now we have nginx-test on our demo ns

![alt text](<images/Screenshot 2024-11-11 at 13.26.59.png>)

We want to see if the pod on the demo ns can reac the one on the default ns.

```
k get pod -n demo
```
Grab the name of the pod and sh inside it:
```
k exec -it nginx-demo-cccbdc67f-tjmkr  -n=demo -- sh
```
![alt text](<images/Screenshot 2024-11-11 at 13.32.33.png>)

From the image above, we are already inside the `nginx-demo-cccbdc67f-tjmkr` pod

we need to get the `IP address` of a pod under the default namespace and see if we can access it through the `pod` under the demo namespace using the `curl` command.
```
k get pods
```
Grab the pod name and check for the IP address

```
k describe pod nginx-dep-84bdb68cd5-5jfjj
```
![alt text](<images/Screenshot 2024-11-11 at 13.40.10.png>)

The `IP Address`is `10.244.1.3`

Inside the Pod under the demo **ns**, use the curl command.

```
curl localhost 10.244.1.3
```
![alt text](<images/Screenshot 2024-11-11 at 13.43.48.png>)

As we can see, we are able to access the other pod under default namespace from the demo namespace. If we try from access pod under the demo ns from the default ns it will be suceessful as well. You can type **exit** to close the terminal.

Let's scale the pod inside the demo ns up to 4
```
k scale deploy nginx-demo --replicas=4 -n demo
```
```
k get pods -n demo
```

![alt text](<images/Screenshot 2024-11-11 at 13.54.21.png>)

Let's create a service for the pod under the demo ns because we will be using a service to communicate with the pod. let create it imperatively.

```
k expose deploy nginx-demo --name svc-demo --port=80 -n demo
```
We use **expose** key word when creating service
```
k get svc -n demo
```
![alt text](<images/Screenshot 2024-11-11 at 14.03.02.png>)

We have a service we created initially under the default ns

![alt text](<images/Screenshot 2024-11-11 at 14.04.43.png>)

Get the name of a pod running insde the default ns and grab the ip address. log inside of it and try to use the `curl` command to access the new service we created under the demo ns.

```
k get pods -o wide
```
![alt text](<images/Screenshot 2024-11-11 at 14.11.57.png>)

```
k get svc -n demo
```
![alt text](<images/Screenshot 2024-11-11 at 14.16.04.png>)

```
k exec -it nginx-dep-84bdb68cd5-5jfjj -- sh
```
```
curl svc-demo
```
![alt text](<images/Screenshot 2024-11-11 at 14.22.43.png>)

From the above image, we know we can access service from another ns. To get the full service address path on each ns we need to cat the following command.
```
cat /etc/resolv.conf
```
The `/etc/resolv.conf` is the file responsible for resolving `IP` address to `dns` reolution internally within the cluster or host.

![alt text](<images/Screenshot 2024-11-11 at 14.30.45.png>)

As we can see we have a different host name we can use. the same thing we happen if we cat the file in the demo ns. We have to follow the **fully qualified domain name** `default.svc.cluster.local` for the **deafult** ns and `demo.svc.cluster.local` for the **demo** ns.

Now if we want to access the service name under `demo` cluster from the `default` cluster we have to follow the domain name.
```
svc-demo.demo.svc.cluster.local
```
![alt text](<images/Screenshot 2024-11-12 at 00.07.50.png>)

Now our pod in one namespace can get a response from a service in another namespace.