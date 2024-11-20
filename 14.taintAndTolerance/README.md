# Taints and Tolerations in Kubernetes

In Kubernetes, **Taints** and **Tolerations** are mechanisms used to control how pods are scheduled on nodes. They provide a way to restrict or allow pods to run on specific nodes, ensuring that workloads are placed appropriately based on the requirements or characteristics of the nodes.

## Taints
A **taint** is applied to a node to indicate that only specific pods (those with matching tolerations) should be scheduled on it.
- Purpose: To repel pods from being scheduled on the node unless they have a matching toleration.
- **Key Use Case:**
    - Reserving a node for specific workloads, such as GPU-intensive jobs or system-level components.
    - Preventing regular application pods from running on nodes dedicated to specific tasks (e.g., storage or logging).

## Tolerations

A **toleration** is applied to a pod to indicate that it can tolerate (or "ignore") specific taints on nodes.
- Purpose: To allow pods to overcome taints and be scheduled on tainted nodes.
- Key Use Case:
    - Allowing specific pods to run on nodes reserved for certain workloads or environments.

**How They Work Together**

- When a node has a taint, only pods with a matching toleration can be scheduled on that node.
- Pods without matching tolerations will avoid or be evicted from that node.

![alt text](<images/Screenshot 2024-11-20 at 11.46.15.png>)

Currently, we have 3 `Nodes`
```
k get nodes
```
![alt text](<images/Screenshot 2024-11-20 at 11.57.03.png>)

Let apply taints to the 2 worker nodes and try to schedule a pod.

The command for tainting a node follow this arrangement.
```
kubectl taint nodes <node-name> key=value:effect
```
```
k taint node cka-cluster3-worker  gpu=true:NoSchedule
```
```
k taint node cka-cluster3-worker2  gpu=true:NoSchedule
```
![alt text](<images/Screenshot 2024-11-20 at 12.05.59.png>)

Let's check the information on the node.
```
k describe node cka-cluster3-worker | grep -i taint
```
```
k describe node cka-cluster3-worker2 | grep -i taint
```
![alt text](<images/Screenshot 2024-11-20 at 12.14.40.png>)

Let try to schedule a pod now
```
k run nginx --image=nginx
```
![alt text](<images/Screenshot 2024-11-20 at 12.16.54.png>)

you can see the pod in pending mode because it doesn't have the toleration to tolerate the node taint. If we describe the pod to further probe it.

```
k describe po nginx
```
![alt text](<images/Screenshot 2024-11-20 at 12.20.07.png>)

You can see it stated that it failed scheduling because, **1 node has untolerated taint(control plane) while the other 2 nodes have untolerated taint {gpu=true}**. The Control plane by default has taint that made it impossible for any pods to be schedule on it.

to resolve it, we either create a toleration on this pod or create a new pod.

Create a new pod and output a yaml because we can not add a `toleration` from the command line.

```
k run redis --image=redis --dry-run=client -o yaml > redis.yaml
```
The format for a taint is like this:
```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```
So let add it to our pod configuration.

![alt text](<images/Screenshot 2024-11-20 at 12.34.35.png>)

```
k apply -f redis.yaml
```
![alt text](<images/Screenshot 2024-11-20 at 12.36.08.png>)

We can see it is running because it has the toleration. To check which node the pod is scheduled:
```
k get po -o wide
```
![alt text](<images/Screenshot 2024-11-20 at 12.38.13.png>)

We can see it is schedule on Node `cka-cluster-worke` while the **nginx** is still in pending mode because of the taint. 

Let remove the taint on node `cka-cluster2-worker`. To remove a taint, follow this format:
```
kubectl taint nodes <node-name> key=value:effect-
```
```
k taint node cka-cluster3-worker  gpu=true:NoSchedule-
```
```
k taint node cka-cluster3-worker2  gpu=true:NoSchedule-
```
```
k get po
```
![alt text](<images/Screenshot 2024-11-20 at 12.45.54.png>)

```
k describe po nginx
```

![alt text](<images/Screenshot 2024-11-20 at 12.47.51.png>)

We can see that the pod has been schedule now on `cka-cluster3-worker` beacuse we have remove the **Taint** on the node.

> [!Note]
> control plane has been tainted by default and it cannot allow any customized pods to be schedule on it. only the **System component** can be schedule on the control plane.
> When a `Node` is tainted it only restricts or stop pods without its tolerance from  scheduling on the node> That does not mean the pods can not be schedule on other nodes. To avoid this we add more features by using **selectors** with the help of labels.

With the help of the selectors and labels we let the pod know which node it can be schedule.

Let create a new pod to demonstrate it.
```
k run new-nginx --image=nginx --dry-run=client -o yaml > new-nginx.yaml
```
```
k apply -f new-nginx.yaml
```
```
k get po
```

![alt text](<images/Screenshot 2024-11-20 at 13.18.12.png>)

```
k describe po new-nginx
```
![alt text](<images/Screenshot 2024-11-20 at 13.19.34.png>)

We can see the pod is pending and when probed further we see that it complain that ** it didn't match any pod/node selector affinity**.

Let's add the label to a node and see if it will be schedule on that node.

```
k label node <nodeName> <label>
```
```
k label node cka-cluster3-worker gpu=false
```
```
k get node --show-labels
```
![alt text](<images/Screenshot 2024-11-20 at 13.27.20.png>)

```
k get pod -o wide
```
![alt text](<images/Screenshot 2024-11-20 at 13.29.22.png>)

We can see that the pod is running now.

So basically, the difference between **Taint/Toleration** and **nodeSelector** is that the former restrict which pod can be schedule on it while the later chose which node it want to be schedule on. The limitation to **nodeSelector** is that we can not use some expression or logic with and we can not add more condition to it so that it can schedule on one or more nodes. We use some concept in kubernetes to tackle that which is **Node Affinity**.