# Static Pods, Manual Scheduling, Labels and Selector

## Static Pods

![alt text](<image/Screenshot 2024-11-16 at 17.23.09.png>)

In Kubernetes, **Static Pods** are pods that are directly managed by the Kubelet on a specific node, rather than being managed by the Kubernetes API server. Static Pods are often used in specific situations like deploying essential system components (e.g., kube-apiserver, etcd) or for debugging and testing. Static pods are stored in one directory and kubelete just monitor that directory. 

Static Pods are directly managed by the Kubelet and are ideal for running node-specific, critical workloads. They bypass the Kubernetes API server and are not managed by the control plane, making them useful for essential system services but less flexible for general application workloads.

> [!NOTE]
> In this study we are making use of **Kind Cluster** which stands for **Kubernetes in Docker**. That means all our nodes are are running in a container. If we are to be using a cloud provided kubernetes, it will be a server that we can `ssh` in and connect with it. But with the`Kind` cluster we are going to `exec it` into it like container.

```
k get nodes
```
```
docker exec -it cka-cluster3-control-plane bash
```
![alt text](<image/Screenshot 2024-11-16 at 16.31.05.png>)

Let check if kubelete is running on the control plane. We do this by check for the running processes and pipe it with grep
```
ps -ef| grep kubelet
```

![alt text](<image/Screenshot 2024-11-16 at 16.36.07.png>)

We can see from the image above that the kubelete is running on the control plane. To check for where the kubelet manifest is store, we check the default manifest directory which **etc/kubernetes/manifests**. This is where where all yaml manfifest of kubernetes are stored.

```
etc/kubernetes/manifests/
```
```
ls -ltr
```
![alt text](<image/Screenshot 2024-11-16 at 16.44.23.png>)

We can see all the yaml file configuration of our `master node`. Kubelet is monitoring all the yaml configuration in this directory. For example let remove one of the file like **kube-scheduler** which is responsible for scheduling pod on nodes. move it to a temporary file

```
mv kube-scheduler.yaml /tmp
```
![alt text](<image/Screenshot 2024-11-16 at 16.50.00.png>)

We can see kube-scheduler has been removed from the directory now.

Let check for the pods running in the `kube-system` namespace. Remember we mention that all the resources are running as pods because we are using **Kind**.

```
k get po -n kube-system | grep scheduler
```
```
k get po -n kube-system 
```

![alt text](<image/Screenshot 2024-11-16 at 16.56.37.png>)

We can see we have `coredns`, `control-plane`, `kube-proxy` and all other component running but `kube-scheduler` has been removed. IF this happen, the `Api-server` or `control-plane` will not stop working but the task that **KUbe-scheduler** is suppose to perform which is scheduling pod on nodes **EXCEPTS** if the **NODE** has been specified in the configuration yaml of the pod when creating it. Exusting pod will continue to run except if they fail.

Let's try to create a pod now

```
k run nginx --image=nginx
```
![alt text](<image/Screenshot 2024-11-16 at 17.09.09.png>)

We can see the pod in the pending state because the scheduler is notworking. If we decribe the pod to probe further.

```
k describe po nginx
```
![alt text](<image/Screenshot 2024-11-16 at 17.11.48.png>)

We can see that **Node** is `None` and the **Status** is `pending` which shows something is running with the scheduler. If we move the kube-scheduler file back to the directory which is the current directory signifying by the dot `.`.

```
mv /tmp/kube-scheduler .
```
```
ls -lrt
```
![alt text](<image/Screenshot 2024-11-16 at 17.17.26.png>)

As soon as the **kubelet** notice that we have the `kube-scheduler` yaml configuration file at the right directory it will start it. Now let check our pod.
```
k get po
```
```
k get po -n kube-system | grep scheduler
```
![alt text](<image/Screenshot 2024-11-16 at 17.20.58.png>)

We can see that the `kube-scheduler` and the `nginx` pods are running now.

Components like **schedular**, **Apiserver**, **Controller manager**, **etcd** are all static pods. If we need to restart any of them we know we can find the configurations in the manifest directory inside the **Control-plane node** and **etc/kubernetes/manifests/**.

## Manual Schedulling

**Manual Pod Scheduling** in Kubernetes refers to the process of assigning a pod to a specific node manually, instead of relying on the Kubernetes scheduler to decide where the pod should run. Normally, Kubernetes automatically schedules pods based on resource availability and other constraints, but with manual pod scheduling, you explicitly control which node runs the pod.

![alt text](<image/Screenshot 2024-11-18 at 11.21.05.png>)

When a pod creatiom initiation is made the `scheduler` will look for a pod with no `Nodename` attached to work on. If a pod as been assigned a node in the configuration file, the scheduler can not work on it. This is a `manual pod scheduling`. For practical let create a pod. Because we have an existing nginx pod we need to delete that first before creating another one.

```
k delete po nginx
```
```
k run nginx --image nginx -o yaml > pod.yaml
```
![alt text](<image/Screenshot 2024-11-18 at 11.50.13.png>)

Remove all the part we don't need yet in the configuration up to the image shown below.

![alt text](<image/Screenshot 2024-11-18 at 11.53.36.png>)

Get the Nodes name and specify the pod to be scheduled on it in the configurstion.
```
k get nodes
```
Let assigned it to `cka-cluster3-worker` node and we are also saying scheduler should not do anything.

![alt text](<image/Screenshot 2024-11-18 at 12.01.02.png>)

We need to delete the `pod` first because one a `node` has been assigned to a pod we cannot change it. We have to delete it first and then create it again.

```
k delete po nginx
```

First, let stop the scheduler from working
```
 k get po -n kube-system | grep scheduler
```
![alt text](<image/Screenshot 2024-11-19 at 18.24.32.png>)

```
docker exec -it cka-cluster3-control-plane bash
```
Go to /etc/kubernetes/manifests/
```
cd etc/kubernetes/manifests/
```
```
ls -ltr
```
```
mv kube-scheduler.yaml /tmp
```
![alt text](<image/Screenshot 2024-11-19 at 18.40.57.png>)

If we try to check for the `scheduler` now we will find out it is not working.

![alt text](<image/Screenshot 2024-11-19 at 18.39.50.png>)

From the example we did before, without the scheduler the pod is suppose to hang in pending mode. Let's create the pod now

```
k apply -f pod.yaml
```
```
k get pod
```

To get the node where the pod is running
```
k get pod -o wide
```
![alt text](<image/Screenshot 2024-11-19 at 18.54.45.png>)

We can see that the pod is running on `cka-cluster3-worker`.

Move scheduler back from /tmp directory
```
cd etc/kubernetes/manifests/
```
```
ls -ltr
```
```
mv /tmp/kube-scheduler.yaml .
```

Check for scheduler back

```
k get pod -n kube-system | grep scheduler
```
![alt text](<image/Screenshot 2024-11-19 at 19.58.11.png>)

