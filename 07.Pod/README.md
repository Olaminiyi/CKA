# Pod in kubernetes
## Imperative VS Declarative

Creating pods through imperative means using tool like kubectl. e.g.

- Create an nginx pod through kubectl
```
kubectl run nginx-pod --image=nginx:latest
```

![alt text](<images/Screenshot 2024-11-02 at 16.11.35.png>)

Creating a pod through a declarative means creating it through Yaml ot Json syntax. We will be using Yaml syntax which is the most popular way of creating it.

Creat a Yaml file `pod.yaml` to store the pod configuration inside your working directory
```
touch pod.yaml
```
The simple configuration for a pod will look like this:

![alt text](<images/Screenshot 2024-11-02 at 16.25.12.png>)

The Pod can be created using kubectl command
```
kubectl apply -f pod.yaml
```
When you run this command it will throw an error because we have an nginx-pod created using imperative method before. We have to delete it first and rerun the apply command again.

Delete a pod using this command
```
kubectl delet pods nginx-pod
```
![alt text](<images/Screenshot 2024-11-02 at 16.32.34.png>)

To troubleshoot a pod, let introduce an error to the configuration of the pod.

![alt text](<images/Screenshot 2024-11-02 at 16.36.40.png>)

IF we run the apply command again we will get an error when checking for the status of the pod.

```
kubectl get pods
```

![alt text](<images/Screenshot 2024-11-02 at 16.38.54.png>)

The status shows imagePullBackoff which is showing the could not be pull due to the error we introduced into the name of the image.

To troubleshoot it, run this command
```
kubectl describe pod nginx-pod
```

![alt text](<images/Screenshot 2024-11-02 at 16.42.29.png>)

> [!Error]
> pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed

We can directly update the content of the pod on the terminal using this command.
```
kubectl edit pod nginx-pod
```
It will be show pod editted and I don't need to use the apply method again before it work.

Another way to troubleshoot it is to get inside the pod using `exec -it` command.
```
kubectl exec -it nginx-pod -- sh
```
![alt text](<images/Screenshot 2024-11-02 at 16.54.03.png>)

This is a simple Yaml file, but in a situation when we have a large yaml configuration with many labels is not easy to write. The easiest way is to write it imperatively and output it into a file.

We can use dry run which will not apply the changes but shows the resources has been created.
```
kubectl run nginx --image=nginx --dry-run=client
```
![alt text](<images/Screenshot 2024-11-02 at 17.00.48.png>)

To show the output on the terminal in a yaml syntax we need to add `o yaml` to the end of the command.

kubectl run nginx --image=nginx --dry-run=client  -o yaml

![alt text](<images/Screenshot 2024-11-02 at 17.03.35.png>)

To save it in a file we just need to forward it into a file.
```
kubectl run nginx --image=nginx --dry-run=client  -o yaml > pod-new.yaml
```
This will create a new pod-new yaml file in the working directory and you can change what you don't want in the configuration.