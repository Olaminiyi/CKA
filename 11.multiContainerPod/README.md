# Multi Container Pod Kubernetes - Sidecar vs Init Container

![alt text](<images/Screenshot 2024-11-12 at 16.24.45.png>)

In Kubernetes, **multi-container pods** allow multiple containers to run together in a single pod. This is useful when containers need to work closely and share `resources`, as they will share the same network and storage. Two common patterns in multi-container pods are `sidecar containers` and `init containers`, each serving a different purpose.

## Sidecar Containers

**Purpose**: A sidecar container is a helper container that runs alongside the main application container within the same pod, often to enhance or extend its functionality.

**Characteristics**

- Runs Concurrently: Sidecar containers start and run alongside the main container(s) for the duration of the pod’s lifecycle.
- Use Cases: Logging, monitoring, or providing network proxies (e.g., handling network traffic).
- Example: Suppose you have a main application container in a pod. You could add a sidecar container to handle logging by collecting, processing, and forwarding logs to an external storage system.

**Benefits**

- Sidecars can help offload certain tasks from the main container, like log processing or file syncing.
- They enable modular design, allowing each container to focus on specific tasks.

**Drawback**

- Since sidecars run throughout the pod’s lifecycle, they can consume resources continuously, which may not be ideal for one-time tasks.

## Init Containers

**Purpose:** An init container is designed to perform setup or initialization tasks before the main application container(s) start.

**Characteristics**

- Runs Sequentially: Init containers run before any main containers and complete their tasks before the main containers start. They only run once, in sequence, and Kubernetes waits for them to finish successfully.
- Use Cases: Performing checks, downloading dependencies, or setting up configurations required by the main container.
- Example: Suppose the main application container relies on a specific configuration file. An init container can be used to fetch or generate this file before the main container starts.

**Benefits**

- Init containers are ideal for pre-conditions or setup tasks that only need to happen once.
- They can simplify main containers by handling setup tasks independently.

**Drawback**

- Init containers don’t persist throughout the pod’s lifecycle, so they are not suitable for ongoing support tasks (like log processing).

For practical purpose, create a pod.yaml file in your working directory with this configuration.
```
apiVersion:  v1
kind:  Pod
metadata:
  name:  myapp
  labels:
    name:  myapp-pod
spec:  
  containers:
  - name:  myapp-containers
    image:  busybox:1.28
    command:  ['sh','-c', 'echo the app is running && sleep 3600']
    env:
    - name:  FIRSTNAME
      value:  Rashford
  initContainers:
  - name:  init-myservice
    image:  busybox:1.28
    command:  ['sh' , '-c']
    args:  ['until nslookup myservice.default.svc.cluster.local; do echo waiting for the service to be up; sleep 2; done']
```
 We are using `environment variables` inside the container to specify some attribute like `name` and `value`.

 **Commands** is the commmand you pass into your container to execute and this could be simple `shell` or `UNIX` commands ****While** the **args** are the arguements we pass into the command.  Under the `main container` the `command` and `args` are combined while under the `init container` they are seperated. We can decide to separate or combine it, it works the same way.

 We are going to `create/expose` a service named `myservice` for the main container. In the `main container`, we are running a command to print the app is running && sleep 3600. in the `init container` we are running a command to **nslookup** command to check the `fully qualified Domain Name` (FQDN) of myservice's service for every 2 seconds until the service is up and print out waiting for the service to be up.


 ```
 k apply -f pod.yaml
 ```
 ```
 k get pods
 ```

![alt text](<images/Screenshot 2024-11-12 at 17.43.19.png>)

From the above image, it is showing that we have 1 pod but not ready and initcontainer is not ready too. Lets decribe the pod to get more information.

```
k describe pod myapp
```
![alt text](<images/Screenshot 2024-11-12 at 17.53.34.png>)

From the image above we can see that the init container status is running and checking for myservice's service to be up through `nslookup` WHILE the myapp-container status is `waiting` as a result of `podInitialization` because `initcontainer` is not completed yet. It is waitng for the condition to be satisfied. And because we haven't expose/create the myservice's service `initcontainer` cannot access it via `nslookup`.

We can check the log and pass the name of the init container
```
k logs myapp -c init-myservice
```
![alt text](<images/Screenshot 2024-11-12 at 18.37.11.png>)

We can see from the image above that the `initcontainer` is trying to do `nslookup` on the `FQDN` of the hostname every 2 seconds.

Let create a deployment

```
k create deploy nginx-deploy --image=nginx --port 80
```
```
k get deploy
```
```
k get po
```
```
kubectl expose deploy nginx-deploy --name myservice --port 80
```

![alt text](<images/Screenshot 2024-11-12 at 18.46.28.png>)

Now the service is exposed now, let check our pods now. It can take some few minutes to initialise

```
k get po
```
![alt text](<images/Screenshot 2024-11-12 at 18.48.17.png>)

We can see our pod is ready now and the `initcontainer` is running. Now let us check the logs of the our pod now.

```
k logs myapp -c init-myservice
```
![alt text](<images/Screenshot 2024-11-12 at 18.51.53.png>)

We can see that it was waiting for the myservice to be up and once the service was up it return the `Fully Qualified Domanin Name` of the service.

We can use this command to print enviroment variables 

```
k exec -it myapp -- printenv
```

![alt text](<images/Screenshot 2024-11-12 at 18.57.54.png>)

We can also sh into the container and print out the enviroment variable.

```
k exec -it myapp -- sh
```

![alt text](<images/Screenshot 2024-11-12 at 19.02.19.png>)

We can also create multi initcontainer in a pod by adding another initcontainer to the configuration.

![alt text](<images/Screenshot 2024-11-12 at 19.06.09.png>)

It will throw an error if make the changes now, we need to delete the pod first and re-create it to apply the change.

```
k delete pod myapp
```
```
k apply -f pod.yaml
```
```
k get po
```

![alt text](<images/Screenshot 2024-11-12 at 19.10.40.png>)

We can see one pod is initialized already while the other is not yet because it is waiting for `mydb` service to comes up.

> [!NOTE]
> For a pod or application that has an initcontainer to be `READY` all the `initcontainer` must be  `completed` or `Running` first.

Let create a new deployment for that will be expose through `mydb`

```
k create deploy mydb --image redis --port 80
```

Let create a service for mydb

```
k expose deploy mydb --name mydb --port 80
```
```
k get svc
```
![alt text](<images/Screenshot 2024-11-12 at 19.19.34.png>)

Our 2 services are ready now.

```
k get pod -w
```
![alt text](<images/Screenshot 2024-11-12 at 19.21.13.png>)


