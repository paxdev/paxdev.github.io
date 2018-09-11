---
layout: post
title: Provisioning Kubernetes pod resources using Helm
---

## in the `values.yaml` file, there is an entry
```
resources: {}
```
This allows you specify values for CPU and memory:

(https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)[https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/]

(https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)[https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/]

Memory is specified in bytes (can be M or Mi, or K or Ki etc.). CPU is specified in a fraction
of a "CPU Unit" where 1 CPU unit = 1 AWS vCPU. They can additionally be specified using _m_ as a unit
to represent a "milliCPU" (1/1000 CPU) so 100m = 0.1 CPU.

The resources have 2 values that can be configued, a **request** and a **limit**.

When provisioning your pods, Kubernetes will look at the total CPU units and memory it has
available for the cluster/namespace and if your **request** exceeds the available resource, your pod will not be provisioned.

When pods are running, Kubernetes will monitor the **limit**.
The container's memory can exceed the **request**, but it **cannot exceed the limit**.
Once the **limit** is exceed, the container will become a candidate for termination and will restart.
The **CPU limit** acts as a throttle. The container will never be allowed to exceed the CPU limit.
```
resources:
  limits:
   cpu: 150m
   memory: 1024Mi
  requests:
   cpu: 100m
   memory: 512Mi
```
In the above example, when provisioning the pod, Kubernetes will only create the pod if it has 100m CPU and 512Mi
memory available.
However, it will allow the CPU time to grow to 150m before throttling and the memory to grow to 1024Mi before
killing the container.

So, you should set the **requests** values to be the amount you expect the container to use under normal circumstances.
The **limits** should be set to allow your container to have short bursts of activity where they exceed the normal
resources without termination.
Of course, if you are going to exceed the **requests** values for any length of time, then you should consider
horizontal scaling. However the higher **limits** will give extra capacity for a short period whilst your autoscaling fires up.
