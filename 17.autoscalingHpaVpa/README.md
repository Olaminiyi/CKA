# Autoscaling

**Autoscaling** in Kubernetes is the process of automatically adjusting the number of pods or the resources allocated to them based on current demand. This ensures optimal performance and resource usage, helping to maintain service reliability while controlling costs. Kubernetes supports two primary types of autoscaling: **Horizontal Pod Autoscaler** (HPA) and **Vertical Pod Autoscaler** (VPA).

![alt text](<images/Screenshot 2024-11-27 at 00.40.46.png>)

## Horizontal Pod Autoscaler (HPA)

HPA adjusts the number of pod replicas in a deployment, replica set, or stateful set based on observed resource usage or custom metrics. It scales horizontally, meaning it increases or decreases the number of running pods.

**How It Works:**

- HPA monitors metrics such as CPU, memory, or custom application-level metrics through Kubernetes' Metrics Server.
- When the observed metrics deviate from the specified target, HPA adjusts the replica count.

**Key Use Cases:**

- Handling traffic spikes by increasing the number of pods.
- Reducing the number of pods during low demand to save resources.

**Advantages:**

- Dynamically adjusts workload capacity.
- Handles sudden increases in demand effectively.
- Prevents resource wastage by scaling down when demand is low.

## Vertical Pod Autoscaler (VPA)

VPA adjusts the **CPU and memory resource requests and limits** of containers within pods. It scales **vertically**, meaning it increases or decreases the allocated resources to a pod rather than changing the number of pods.

**How It Works:**

- VPA monitors resource usage over time and provides recommendations for appropriate resource requests and limits.
- Depending on the mode (e.g., "Auto" or "Manual"), it can either apply the changes automatically or suggest them for manual implementation.

**Key Use Cases:**

- Optimizing resource allocation for applications with fluctuating resource needs.
- Preventing over-provisioning and under-provisioning of CPU and memory resources.

![alt text](<images/Screenshot 2024-11-27 at 00.44.10.png>)

We will still need the `metric server` to practicalise the HPA. We have the `metric server` already installed from the previous lession.

Create   a yaml file named deploy.yaml with the configuration below. It contains a `deployment` and a `service`.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```




- Spring framework

  - Spring Boot
  - Maven
  - Spring Security
  - Spring Batch
  - Hibernate

- Tomcat

- Ant

- Testing
  - Test Driven Development
  - J-Unit
  - Jasmine
  - Automated test frameworks
  - User testing

- Micro-Service Architecture

- Develop
  - API design and open standards
  - RESTful APIs
  - Swagger
  - Open API

- Security
  - OWASP Top Ten
  - Denial of Service
  - SQL Injection
  - Cross-Site Request Forgery

- High Availability products

- EDB failover manager

- RPC concepts and transport mechanisms
  - HTTP
  - Shared memory

- Experience within the Payments/Banking/FinTech Sector





 char y;
 boolean state;
 byte number;
 float result;
