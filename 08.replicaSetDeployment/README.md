# Replication Controller, Replica Sets and Deployments

Replication controller work is to make sure a we have all the replicas running all the time and make sure that the application does not crashes even if there is a pod faliure.

![alt text](<images/Screenshot 2024-11-04 at 11.03.22.png>)

In case the current `Node` we are using is running out of resources, we can spin up another node. The Replication controller can span across many `Nodes` and maintan the pods on all nodes. 

![alt text](<images/Screenshot 2024-11-04 at 11.07.38.png>) 

**When writing the Yaml configuration of any resources, if you are not sure of its `apiVersion` you can use this command to check. For replication controller use this.

```
kubectl explain rc
```
![alt text](<images/Screenshot 2024-11-04 at 11.17.48.png>)

Creat a rc.yaml in the working directory with the following configuration for replication controller.

```
apiVersion:  v1
kind:  ReplicationController
metadata:
    name:  nginx-rc
    labels:
      env:  demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name:  nginx
    spec:
      containers: 
      - image:  nginx
        name:  nginx
  replicas: 3
```

Apply the configuration by running
```
kubectl apply -f rc.yaml
```
![alt text](<images/Screenshot 2024-11-04 at 12.27.20.png>)

```
kubectl get rc
```

## ReplicaSet

![alt text](<images/Screenshot 2024-11-04 at 13.12.15.png>)

The difference between ReplicationController and ReplicaSet is that the former one is the legacy version while the latter one is the version that is been currently use.  if we are given the option, ReplicaSet is the better choice. Also, ReplicationController can only control the pods that were created within its configuration **WHILE** Replicaset we can manage existing pods that were not part of the ReplicaSet. We do that with the help of another filed called `selector`. Inside the `seletor` we use `matchLabels` to match the name of the pods we want to be part of the ReplcaSet

Create rs.yaml and paste this ReplicaSet configuration.
```
apiVersion:  v1
kind:  ReplicaSet
metadata:
    name:  nginx-rs
    labels:
      env:  demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name:  nginx
    spec:
      containers: 
      - image:  nginx
        name:  nginx
  replicas: 3
  selector:
    matchLabels:
      env: demo

```
Running this will throw an error that `resources mapping not found for  name nginx-rs namespace`. This is showing that Replicaset does not exist in that namespace. To resolve this run this command.
```
kubectl explain rc
```

You will find out that ReplicaSet belongs to a **Group** `apps` before **VERSION** `v1`. so we need to update the `apiVersion` to reflect the `group`
```
apiVersion:  apps/v1
kind:  ReplicaSet
metadata:
    name:  nginx-rs
    labels:
      env:  demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name:  nginx
    spec:
      containers: 
      - image:  nginx
        name:  nginx
  replicas: 3
  selector:
    matchLabels:
      env: demo

```
![alt text](<images/Screenshot 2024-11-04 at 12.52.39.png>)

It shows that ReplicationController's pod is still showing. We can delete ut and check back again.

```
kubectl delete rc nginx-rc
```
```
kubectl get po
```
![alt text](<images/Screenshot 2024-11-04 at 12.56.03.png>)

Now we have 3 Pods.

There are two ways to update or edit the configuration. We can edit directly from the file and rerun the `kubectl apply` command or edit it live using this command.

```
kubectl edit rs nginx-rs
```
![alt text](<images/Screenshot 2024-11-04 at 12.59.43.png>)

You can directly edit it lively and it will be aplly automatically.

lets cahange the replica from 3 to 7 and see what happen.

![alt text](<images/Screenshot 2024-11-04 at 13.03.36.png>)

We can see the pod has increase to 7 in numbers.

Another method is to use this kubeclt command

```
kubectl scale --replicas=10 rs nginx-rs
```
![alt text](<images/Screenshot 2024-11-04 at 13.09.57.png>)

## Deployment

As a user, we create **Deployment** and it will in turn create ReplicaSet While ReplicaSet will manage the pods. Deployment will manage the ReplicaSet, it will create additional functionality.

If we are running an application or a collection of pods on version 1.1 and we want to update it to 1.2 version, if we try to upgrate it using `ReplicaSet` it will upgrade everything at the same time thereby leading to a `downtime` which we cannot afford in `life environment`. But if we run it using `Deployment` it will run it will make the changes in a rolling update version. While one pod is benn updated it will forward all the traffic to the other ones and it can also spin up another pod to manage the load. When the particular pod that is been updated ready it will be added to the `ReplicaSet`.

![alt text](<images/Screenshot 2024-11-04 at 13.26.55.png>)

 It will continue untill all the application are updated without any downtime.

 ![alt text](<images/Screenshot 2024-11-04 at 13.27.36.png>)

 We can always roll back if there is a problem.

We can delete the ReplicaSet we created and see how deployment works.
```
kubectl delete rs nginx-rs
```

Create a new file deploy.yaml in the working directory with this deployment configuration.

```
apiVersion:  apps/v1
kind:  Deployment
metadata:
    name:  nginx-deploy
    labels:
      env:  demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name:  nginx
    spec:
      containers: 
      - image:  nginx
        name:  nginx
  replicas: 3
  selector:
    matchLabels:
      env: demo

```

![alt text](<images/Screenshot 2024-11-04 at 13.34.10.png>)

```
kubectl get deploy
```

If we do kubectl get all it will return all the objects running in the cluster

```
kubectl get all
```

After running the `kubectl get all`, it shows we have

1. 3 pods ready and running
2. 1 service which is a default service of our cluster
3. 1 deployment with 3 ready and running
4. 1 replicaset with 3 ready and running

If we intend to change the version of our image to another, we could achieve it using this command.
```
kubectl set image deploy nginx-deploy \
nginx=nginx:1.9.1
```
```
kubectl describe deploy nginx-deploy
```
![alt text](<images/Screenshot 2024-11-04 at 13.48.03.png>)

![alt text](<images/Screenshot 2024-11-04 at 13.48.52.png>)

We can see that the image has been updated to 1.9.1. The configuration in our yaml file willl not be updated because we updated it on life.

Now if we want to check the rollout history for our deployment

```
kubectl rollout history deploy/nginx-deploy
```
![alt text](<images/Screenshot 2024-11-04 at 13.53.31.png>)

We can also rollback our deployment 

```
kubectl rollout undo deploy/nginx-deploy
```

![alt text](<images/Screenshot 2024-11-04 at 13.56.44.png>)

So if we describe the dployment now we will see it has been rollback to the previous version.

![alt text](<images/Screenshot 2024-11-04 at 13.59.32.png>)

To make it easier we can use our kubectl command with dry-run and output it into a yaml file. For example;
```
kubectl create deploy deploy/nginx-new --dry-run=client --image=nginx -o yaml > new-deploy.yaml
```
![alt text](<images/Screenshot 2024-11-04 at 14.06.48.png>)

You can open the yaml file and update it your requirement.

![alt text](<images/Screenshot 2024-11-04 at 14.08.18.png>)
