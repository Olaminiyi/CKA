# Node Affinity

In Kubernetes, **Node Affinity** is a mechanism that controls which nodes a pod can or should be scheduled on, based on specific rules. It allows you to express constraints for pod scheduling in a more flexible and expressive way than basic nodeSelector.

![alt text](<images/Screenshot 2024-11-20 at 23.44.04.png>)

![alt text](<images/Screenshot 2024-11-20 at 23.45.00.png>)

## How Node Affinity Works

Node affinity relies on labels assigned to nodes. By using these labels, you can define rules for pods that dictate where they should or should not be scheduled.

**For example:**

- You might label nodes with disktype=ssd to indicate they have SSD storage.
- Then, you can create a pod that specifies it should only run on nodes with the disktype=ssd label.

## Types of Node Affinity

1. **RequiredDuringSchedulingIgnoredDuringExecution:**

This is a hard requirement for scheduling. If a node doesn't meet the condition, the pod will not be scheduled on it.

**Behavior:** The rule is enforced only during scheduling. Once the pod is running, it won't be evicted if the node’s labels change.

2. **PreferredDuringSchedulingIgnoredDuringExecution:**

This is a soft preference. Kubernetes tries to place the pod on a node that meets the condition but will still schedule it elsewhere if no matching nodes are available.

**Behavior:** Like the required type, this is only applied at scheduling time and not afterward.

**Node Affinity vs Node Selector**

- **Node Selector:** Matches pods to nodes using exact label matches. Simple but lacks flexibility.

- **Node Affinity:** More expressive, allowing operators like `In`, `NotIn`, or `Exists` to create more complex scheduling rules.

## Benefits of Node Affinity

- More Flexible Scheduling: Supports advanced rules, unlike the simpler nodeSelector.
- Improved Resource Management: Ensures pods are placed where they can use resources effectively.
- Operational Efficiency: Helps schedule pods based on node-specific features like storage type, GPU availability, or geographic location.

For practical, lets create a pod, create affinity.yaml in the working directory. The content loook like this:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd


```
```
k apply -f affinity.yaml
```
![alt text](<images/Screenshot 2024-11-21 at 00.16.54.png>)

```
k describe po redis
```

![alt text](<images/Screenshot 2024-11-21 at 00.19.26.png>)

We can see that the `pod` is in a pending state and this is as a result of the `Node` not match the selector that matches the affinity in the pod configuration.

Let apply label to one of the nodes

```
k get nodes
```
```
k label node cka-cluster3-worker disktype=ssd
```
![alt text](<images/Screenshot 2024-11-21 at 00.26.08.png>)

```
k get nodes --show-labels
```
![alt text](<images/Screenshot 2024-11-21 at 00.27.43.png>)

Let check the pod now
```
k get po -o wide
```
![alt text](<images/Screenshot 2024-11-21 at 00.30.21.png>)

Let create another pod with another type of **Node affinity**.

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis2
spec:
  containers:
  - image: redis
    name: redis2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  affinity:
    nodeAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: label-1
                operator: In
                values:
                - key-1


```
```
k apply -f affinity2.yaml
```

![alt text](<images/Screenshot 2024-11-21 at 00.39.01.png>)

We can see the pod running even if we do not have any matching selector label for the affinity on the pod. This is because the type of `Node affinity` we are using is ** preferredDuringSchedulingIgnoredDuringExecution** which does not mandated the node to have the label selector label of the affinity before the pod can be schedule on it. If the pod found a node with the selector label that's `fine` and if not it will still be schedule on the node.

> [!NOTE]
If we make any changes to the node the `running pods` does not get affected except for the new pod that is about to be created.

Let's delete the selector label from the node we added it to the previuos time.

```
k label node cka-cluster3-worker disktype-
```
```
k get po
```
![alt text](<images/Screenshot 2024-11-21 at 00.49.51.png>)

We can see the pod is still running.

There are many operator we can use with `node affinity` such as In, NotIn, or Exists to create more complex scheduling rules. 

Let's check the `Exist` operator and see how it works. Let's make use of the affinity.yaml file and edit it to the configuration of `Exist` operator.

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis3
spec:
  containers:
  - image: redis
    name: redis3
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: Exists
            
```

Let give one of the nodes a label of `disktype`

```
k label node cka-cluster3-worker disktype=
```
```
k get nodes --show-labels
```

![alt text](<images/Screenshot 2024-11-21 at 00.59.10.png>)

```
k apply -f affinity.yaml
```
15.nodeAffinity/images/Screenshot 2024-11-21 at 01.05.11.png

We can see that the pod is running and schedule on the `cka-cluster3-worker` using the `Exists` pod affinity.

We make use of both **Taint/Toleration** and **Node** affinity together to make sure `Nodes` are only accomodating pod that specifically made for them. 