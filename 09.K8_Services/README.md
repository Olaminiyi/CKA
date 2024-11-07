#  Kubernetes Services Explained - ClusterIP vs NodePort vs Loadbalancer vs External

## NodePort

A **NodePort** is a way to expose a Kubernetes service on each Node’s IP address at a static port, which is typically in the range of 30000–32767. This allows external traffic to access your application through the nodes in the cluster.

### How Does It Work?

 When you create a service with type: `NodePort`:

Kubernetes automatically selects a port (or you can specify one) from the range 30000–32767 and opens it on all nodes in your cluster.
This NodePort maps to a specific port in the service’s ClusterIP, which is how Kubernetes routes traffic inside the cluster.

In a simpler term, we will have a NodePort that we can select or be selected for by the kubernetes (30000–32767) which the outside service or user can connect to to interact with our application. Then, there is a port that we can refer to as a `port` or `ClusterIP:Port`that will serve as a link between the `NodePort` and the port which is refer to as `targetPort` of the application we want to expose. The inetermediate port i refer to as `ClusterIP:Port` will be a point of connection between our `targetPort` and any other application within the cluster that want to communicate with it.

![alt text](<images/Screenshot 2024-11-06 at 16.17.40.png>)

So the traffic goes

```
External Request (NodeIP:NodePort) -> Any Node in the cluster -> ClusterIP:Port -> Target Pod(s)

```
Let create a NodePort service for practical. Create a nodeport.yaml in your working directory

You can use the following command to check the name of your pod and get the label info

```
kubectl get po
```
```
kubectl get pod --show-labels nginx-deploy-987c75f7-dz9lf
```

![alt text](<images/Screenshot 2024-11-06 at 16.39.32.png>)

put this NodePort configuration in the file we created.

```
apiVersion:  v1
kind:  Service
metadata: 
  name:  nodeport-svc
  labels:
    env: demo
spec:
  type:  NodePort
  ports:
  - nodePort: 30001
    port:  8080
    targetPort:  80
  selector:
    env:  demo 
```
```
bubectl apply -f nodeport.yaml
```
```
kubectl get svc
```
we describe the pod to get know the Node its running on

```
kubeclt get po
```
```
kubectl describe pod nginx-deploy-987c75f7-dz9lf
```

![alt text](<images/Screenshot 2024-11-06 at 17.09.08.png>)

We can see our pod/application is running on `cka-cluster2-worker2/172.18.0.5`. The IP Address is `172.18.0.5`.

We can also get the IP adress of the node because we are exposing the application through the Node using the command below.

```
kubectl get node -o wide
```
![alt text](<images/Screenshot 2024-11-06 at 17.11.30.png>)

Our pod is running on IP: 172.18.0.5 and we should be able to view it via 172.18.0.5:30001 But because we are using `KIND` does not expose NodePort service externally. The way Nodeport works on it is different. We still need to do some configuration when creating the cluster. 

So will will recreate a new cluster adding new new configuration so our Nodeport can work perfectly. We need to delete the cluster we are working on. 

Create a config file and with this configuration for our new cluster;
```
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30001
    hostPort: 30001
- role: worker
- role: worker
```

Create a new cluster with the command below using the configuration setting in the config.yaml file.

``
kind create cluster --config config.yaml --name cka-cluster3
``

Creat a new deployment imperatevely through kubectl using `nginx` **image**
```
kubectl create deployment nginx-dep --image nginx --port=80 --dry-run=client -o yaml > nginx.yaml
```

Re-create NodePort service and make sure the matchlabel is matching the new deployent `nginx` label
```
kubectl apply -f nodeport.yaml
```
Get the service created and check the `port` on which is running 

```
kubectl get svc
```
![alt text](<images/Screenshot 2024-11-06 at 23.56.19.png>)

From the image above you can see the service is running on port 8080:30001.

You can try it with `Curl` command on your local machine or `localhost:30001`

```
curl localhost:30001
```
![alt text](<images/Screenshot 2024-11-07 at 00.01.42.png>)

![alt text](<images/Screenshot 2024-11-07 at 00.03.06.png>)

Because we are using Kind that is why Nodeport is been accessed via `localhost`. If it were to be kubernetes it will be accessed via the `Node` IP address the pod is running on.

> [!NOTE]
> To create a shortcut for some commands on your local machine.
```
cd ~
```
```
vi .bash_profile
```
Edit it 
```
alias 'k=kubectl'
```
Exit
```
source .bash_profile
```

## ClusterIP

ClusterIP is a type of service that provides an internal IP address for communication within the cluster. It is used to allow pods within the same Kubernetes cluster to communicate with each other but is not accessible from outside the cluster. This service is created for the purpose of connecting one application/pod/component to another to communicate internally within the cluster.

![alt text](<images/Screenshot 2024-11-07 at 12.24.27.png>)

Create a clusterip.yaml file for our configuration. The configuration is similar to the Nodeport but we will change the type to `ClusterIp` or remove the type entirely. clusterIp is a default service type in kubernetes, so it will create it without indicating it.
```
apiVersion:  v1
kind:  Service
metadata: 
  name:  cluster-svc
  labels:
    env: demo
spec:
  ports:
  - port:  8080
    targetPort:  80
  selector:
    env:  demo 
    pod-template-hash:  84bdb68cd5

```
```
k apply -f clusterip.yaml
```
```
k get svc
```
![alt text](<images/Screenshot 2024-11-07 at 12.54.30.png>)

We can see we have two ClusterIP now, one for default kubernetes and the new clusterip service we created.
```
k describe svc clusterip-svc
```
![alt text](<images/Screenshot 2024-11-07 at 12.58.05.png>)

If you notice the `Endpoint`, they are the ip addresses that the service are listening or running on. You can run the command below to get endpoint for all resources running on your cluster.
```
k get ep
```
![alt text](<images/Screenshot 2024-11-07 at 16.56.51.png>)


## LoadBalancer

![alt text](<images/Screenshot 2024-11-07 at 17.32.42.png>)

In a Kubernetes cluster, a `LoadBalancer` is a resource type that exposes your services to the internet, distributing incoming network traffic evenly across multiple instances of an application (or "pods") within the cluster. This improves application performance, reliability, and availability. When you create a service of type LoadBalancer, Kubernetes interacts with the cloud provider (e.g., AWS, Azure, GCP) to provision an external load balancer. The cloud provider automatically assigns an external IP address to the service, enabling clients outside the Kubernetes cluster to reach your application. Create a lb.yaml file in your working directory for a LoadBalancer configuration or we can create it imperately via kubectl with the command below.
```
kubectl expose deploy nginx-dep  --type=LoadBalancer --name=lb-svc --port=8080 --target-port=80 --dry-run=client -o yaml > lb.yaml
```
```
k apply -f lb.yaml
```
```
k get svc
```
![alt text](<images/Screenshot 2024-11-07 at 17.30.07.png>)

You can see from the image above that we don't have an IP address for the loadbalanacer. That's because we have just created it as a service but not provision it as aload balancer yet. It will operate as a NodePort service now and it can be acessed via this **port** `8080:30001/TCP` 

## ExternalName

In Kubernetes, an ExternalName service is a special type of service that allows you to map a service to an external DNS name rather than providing access to an internal set of pods. This type of service is useful for integrating external, non-Kubernetes resources with Kubernetes workloads by giving them a Kubernetes service endpoint.

### How ExternalName Services Work:

- An ExternalName service doesn’t have a ClusterIP or load balancing, and it does not directly route traffic to Kubernetes pods.
- Instead, it acts as an alias or redirect to an external domain name (a fully qualified domain name, or FQDN).
- When a request is made to the ExternalName service, it forwards the request to the external DNS name specified.

### Use Cases for ExternalName Services:

- External Database or Service Integration: If you have an external database or service hosted outside your Kubernetes cluster, you can use an ExternalName service to connect it seamlessly without modifying your application code.
- Service Aliasing: If you want applications within Kubernetes to reference an external service with a simple name (e.g., external-db), you can define an ExternalName service to route traffic to that service without changing the external service’s DNS.

We an creat an ExternalName service with the configuration below

```
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: example.com

```