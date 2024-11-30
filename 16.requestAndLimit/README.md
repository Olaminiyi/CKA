# Kubernetes Request and Limit

In Kubernetes, **requests and limits** are part of resource management, used to control how much `CPU` and `memory (RAM)` a container can use. These settings help ensure fair resource allocation and prevent any single container from overusing the node's resources, which could affect other workloads

![alt text](<images/Screenshot 2024-11-22 at 11.37.42.png>)

![alt text](<images/Screenshot 2024-11-22 at 11.38.31.png>)

## Resource Request

This is the minimum amount of CPU or memory guaranteed to a container.
- When scheduling a pod, Kubernetes checks if the node has enough available resources (CPU/memory) to satisfy the request.
- If the node doesn't meet the request, the pod won't be scheduled there.
- Once running, the container is allowed to use more than the requested resources if other resources are available.

## Resource Limit

The maximum amount of CPU or memory a container is allowed to use.
- Kubernetes ensures that the container cannot use more resources than the specified limit.
- If the container tries to exceed the limit:
    - For CPU: The container is throttled (its CPU usage is reduced).
    - For Memory: The container is terminated (killed) to free up memory.

![alt text](<images/Screenshot 2024-11-22 at 11.40.21.png>)

## How Requests and Limits Work Together
- Requests: Define resource guarantees for scheduling purposes.
- Limits: Define resource usage boundaries to prevent overconsumption.

To practicalise it we need to create a **metric server first**.

**Metrics Server** is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines.

Metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through Metrics API for use by **Horizontal Pod Autoscaler** and **Vertical Pod Autoscaler**. Metrics API can also be accessed by **kubectl top**, making it easier to debug autoscaling pipelines.

The metric server will expose the metrics of our nodes. check the content inside the metric-server.yaml.

```
k apply -f metric-server.yaml
```
![alt text](<images/Screenshot 2024-11-22 at 12.01.13.png>)

We will not go into details about some content in the metric-server.yaml because it will be covered later.

The metric server we created will be running inside the `kube-system` namespace because it is a `kubernetes add-ons`, it will be running inside the `Control plane`.

```
k get po -n kube-system
```
![alt text](<images/Screenshot 2024-11-22 at 12.05.29.png>)

If we run this command 
```
k top node
```
it will show our nodes metrics with the help of metrics-server we created earlier. 

![alt text](<images/Screenshot 2024-11-22 at 12.08.03.png>)

We can now use this information to create a `stress test` for our nodes.

Create a new name space and create some pods inside the new namespace

```
touch memory-request-limit.yaml memory-request-limit-2.yaml memory-request-limit-3.yaml
```

```
k create namespace mem-example
```

The images use in the pods we want to create in `memory-request-limit.yaml`, `memory-request-limit-2.yaml` and `memory-request-limit-3.yaml` are buid specifically to stress the nodes base on the argument we pass in to it. **Note** also that request can be of `memory` and `cpu` not only `memory`.

For the `memory-demo` pod

![alt text](<images/Screenshot 2024-11-22 at 12.16.00.png>)

`100Mi` was the memory requested and that it can also reach a limit of `200Mi` if needs more memory. The args section in the configuration file provides arguments for the Container when it starts. The "--vm-bytes", "150M" arguments tell the Container to override the limit up to 150 MiB of memory.

There are 2 ways to go when trying to create a resources in a namespace. We either create it imperativley by signifying the `name` of the namespace like this;
```
k apply -f memory-request-limit.yaml -n mem-example
```
Or add the namespace in the yaml configuration like this.

![alt text](<images/Screenshot 2024-11-22 at 12.29.33.png>)

```
k apply -f memory-request-limit.yaml
```
```
k get po -n mem-example
```
```
k top pod memory-demo -n mem-example
```
![alt text](<images/Screenshot 2024-11-22 at 12.35.41.png>)

We can see that the pod is running and using `150Mi` of memory. this is because the memory did not exceed the  memory limit of `100Mi` and `200Mi`.

Let do another practical using ` memory-request-limit-2.yaml` configuration.

![alt text](<images/Screenshot 2024-11-22 at 12.16.45.png>)

In the args section of the configuration file, you can see that the Container will attempt to allocate `250 MiB` of memory, which is well above the `100MiB` limit.

```
k apply -f memory-request-limit-2.yaml
```
```
k get po -n mem-example
```
```
k top pod memory-demo-2 -n mem-example
```

![alt text](<images/Screenshot 2024-11-22 at 12.44.42.png>)

We can see the pod crashing with status `CrashLoopBackOff`

```
k describe  memory-demo-2 -n mem-example
```

![alt text](<images/Screenshot 2024-11-22 at 12.50.33.png>)

![alt text](<images/Screenshot 2024-11-22 at 12.50.05.png>)

We can see the pods is trying to pull the image but the image keep crashing and when checked in the second image is as a result of `OOM` which is `Out Of Memory`. So instead of killing the entire node, it was the pod that was killed to allow another node schedule on it.

Now using the third file `memory-request-limit-3.yaml`, we set the memory limit more than the node capacity and overriding it with `150Mi`.


```
k apply -f memory-request-limit-3.yaml
```
```
k get po -n mem-example
```
```
k describe pod memory-demo-3 -n mem-example
```

![alt text](<images/Screenshot 2024-11-22 at 12.50.33.png>)

![alt text](<images/Screenshot 2024-11-22 at 13.01.13.png>)

We can see from the error message `1 Preemption is not helpful for scheduling, 2 No preemption victims found for incoming pod.`. The 2 workers node are not found because the request is more than their capacity.

Delete the pods.

```
k delete po memory-demo memory-dem
o-2 memory-demo-3 -n mem-example  
```
