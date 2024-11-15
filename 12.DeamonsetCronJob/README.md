# Deamonset, CronJon and Job

## Deamonset

![alt text](<images/Screenshot 2024-11-15 at 00.30.34.png>)

In Kubernetes, a **DaemonSet** is a resource type that ensures a particular pod runs on all (or specific) nodes within a cluster. It’s especially useful for deploying system-level services that need to operate on every node, such as logging, monitoring, or network services. Resources like kube-proxy are deployed on the kubernetes as a deamonset.

## Key Concepts of a DaemonSet:
- Purpose: A DaemonSet guarantees that a copy of a specific pod is running on each node. If a new node is added, Kubernetes automatically deploys the DaemonSet pod on the new node as well.

- Use Cases:
  - Monitoring Agents: Deploying agents like Prometheus Node Exporter or Datadog on every node to gather metrics.
  - Logging Services: Installing logging agents like Fluentd on all nodes to collect and forward logs.
  - Network Services: Running services that require access to the host network, like DNS services or networking plugins.

- How It Works:
  - When a DaemonSet is created, Kubernetes schedules a pod on each eligible node based on the DaemonSet’s specifications.
  - As nodes are added or removed, Kubernetes automatically updates the DaemonSet, adding or removing pods to match the current state of the cluster.

- Selective Deployment:
  - You can configure a DaemonSet to run only on specific nodes by setting node selectors or node affinity rules. This is useful if the DaemonSet service is only needed on certain types of nodes.

- Managing DaemonSets:
  - You can update, delete, or scale a DaemonSet like any other Kubernetes resource, and Kubernetes will ensure that the DaemonSet pods are managed automatically across nodes.

  Creating a Yaml file for a deamonset is similar to the one we created for the deployment. we just need to make few changes to it.
```
touch ds.yaml
```
```
apiVersion:  apps/v1
kind: DaemonSet
metadata:
  name:  nginx-ds
  labels:
    env: demo 
spec:
  template:
    metadata:
      name: nginx
      labels:
        env:  demo
    spec:
      containers:
      - image: nginx
        name:  nginx
        ports:
        - containerPort:  80
  selector:
    matchLabels:
      env:  demo

```
```
k apply -f ds.yaml
```
```
k get po
```
```
k get nodes
```
We can see we have two pods created. From the image above we have 3 nodes, but we know daemonset suppose to create a pod on each node running in a cluster. The reason is because the `control-plane node` is `tainted` so no pod could be schedule to run on it. pod can only be schedule to work on `worker's node`.

Let delete one of the pods.
```
k delete po nginx-ds-r8vns 
```
```
k get po
```
![alt text](<images/Screenshot 2024-11-15 at 01.30.57.png>)

We can see that another pod was created immmediately to replace the deleted one.

To check for resource in all namespaces, use
```
K get <resources> -A
```

We mentioned above that kube-proxy was deploy on the cluster as a daemonset too. It is running on the kube-system namespace. Lets check other namespace to confirm it.

```
k get ds -n kube-system
```
![alt text](<images/Screenshot 2024-11-15 at 01.40.09.png>)

We have 2 `daemonset` on the namespace, the `kube-proxy` and kindnet by `kind` engine for networking. We have 3 `desired`, 3 `ready`. That shows that kube-proxy is deployed on our 3 nodes.

If we describe a pod out of the one we created through the `daemonset`.

```
k get po
```
```
k describe po nginx-ds-qxmwj
```

![alt text](<images/Screenshot 2024-11-15 at 01.46.33.png>)

We can see that the pod is been controlled a a `daemonset` just like checked on the `replicaset`.

## CronJob

In Kubernetes, a **CronJob** is a resource that allows you to schedule and run jobs (tasks) at specified intervals, similar to the cron jobs found in Unix/Linux systems. It is useful for recurring tasks such as batch processing, data cleanup, scheduled backups, and other automated maintenance.

## Key Concepts of a CronJob in Kubernetes:

1. Scheduled Jobs:

- A CronJob runs tasks at specific times, dates, or intervals based on a cron expression.
- Each scheduled execution creates a Kubernetes Job, which runs a single pod to complete the specified task.

2. Cron Syntax:

- A CronJob uses a cron expression to define its schedule. The cron syntax in Kubernetes follows this pattern:

```
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of the week (0 - 7) (Sunday=0 or 7)
│ │ │ └──── Month (1 - 12)
│ │ └────── Day of the month (1 - 31)
│ └──────── Hour (0 - 23)
└────────── Minute (0 - 59)

```
![alt text](<images/Screenshot 2024-11-15 at 01.52.25.png>)

For example, for a job to run at every Saturday by 11pm

```
* 23 * * 6
```
![alt text](<images/Screenshot 2024-11-15 at 02.01.37.png>)

For a job to run at every 5 minutes: */n to every nth interval of time

```
*/5 * * * *
```
![alt text](<images/Screenshot 2024-11-15 at 02.04.22.png>)

3. Running Jobs:
  - Each time a CronJob runs, Kubernetes creates a Job object, which in turn creates one or more pods that execute the task.
  - Each Job runs to completion. Once the Job completes, the pod created by the Job is terminated.

4. Use Cases:
  - Scheduled Backups: Regularly backing up a database at a certain time each day.
  - Data Cleanup: Clearing old logs or data periodically to save storage.
  - Batch Processing: Processing data files or generating reports at specific intervals.

5. Concurrency Policy:
  - CronJobs have concurrency policies that define what happens if a new Job starts while a previous one is still running:
     - Allow: Multiple Jobs can run concurrently (default).
     - Forbid: Prevents new Jobs from starting if the previous one hasn’t finished.
     - Replace: Cancels the currently running Job and replaces it with the new one.

6. Time Zone Awareness:
  - Kubernetes CronJobs do not automatically adjust for time zones. You need to set the cron expression according to the cluster’s default time zone (often UTC), or configure time zone handling within the pod itself.

Example of a cronjob yaml 

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 0 * * *"    # Run every day at midnight
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: my-backup-image:latest
            args:
              - /bin/bash
              - -c
              - "perform-backup.sh"   # Backup script or command
          restartPolicy: OnFailure

```

In this example:

- `schedule: "0 0 * * *"` specifies that the job runs at midnight every day.
- The `jobTemplate` defines the job that will run, with a single container that executes the perform-backup.sh script.
- `restartPolicy`: OnFailure means that if the job fails, it will attempt to restart.