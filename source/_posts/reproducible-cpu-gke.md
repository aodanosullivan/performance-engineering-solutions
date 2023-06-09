---
title: Reproducible CPU Utilization in Kubernetes/GKE
date: 2023-06-09 14:39:51
photos: 
	- /img/posts/cpu.png
tags: [cpu, kubernetes, monitoring]
categories:
 - [cpu]
 - [kubernetes]
 - [monitoring]
---

With the advent of containerized technologies such as Kubernetes it has become increasingly challenging to properly measure the application CPU utilization for performance testing and benchmarking purposes. In this article we will examine how to improve CPU utilization reproducibility between performance tests and therefore make our performance testing and benchmarking more reliable.

# CPU Utilization in GKE - Issues

![img.01](reproducibility0.png)

![img.02](reproducibility1.png)

# Default Limits and Requests Settings for USG

usg\-akamas container resources settings:

limits:            cpu: "11"            ephemeral\-storage: 6Gi            memory: 6Gi          requests:            cpu: 2500m            ephemeral\-storage: 1Gi            memory: 6Gi

# Kubernetes QoS classes

Kubernetes introduces 3 QoS classes: Best Effort\, Burstable and Guaranteed\.

By default if limits are not equal to requests Kubernetes is running the pod in a Burstable class\, which may affect the CPU stability depending on the cluster load etc\.

Documentation: [https://kubernetes\.io/docs/tasks/administer\-cluster/cpu\-management\-policies/\#static\-policy](https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/#static-policy)

# Requirements for the Guaranteed QoS Class

![img.03](reproducibility3.png)

# Adjusting Limits and Requests for the USG Application

Limits and requests settings were made equal for usg\-akamas and cpu was set to an integer value\!

limits:            cpu: "4"            ephemeral\-storage: 6Gi            memory: 6Gi          requests:            cpu: "4"            ephemeral\-storage: 6Gi            memory: 6Gi

# Checking QoS class

$ kubectl describe pod akamas\-usg\-green\-8546ccd7c7\-9p6dk \-n usg\-akamas | grep QoS

QoS Class:                    __Guaranteed__

# Rerunning the Series of Tests

We finally have decent CPU reproducibility between runs \(1\.11\-1\.14 core on average for 8 consecutive runs\)

![img.04](reproducibility2.png)

# Conclusions and Beyond

Setting the Kubernetes limits and requests have crucial impact on your CPU Utilization reproducibility/stability

QoS classes for pods seem to be the crucial concept

This and much more info in documentation: [https://kubernetes\.io/docs/tasks/administer\-cluster/cpu\-management\-policies/\#static\-policy](https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/#static-policy)

To be checked: CPU static policy \(requires Kubernetes version 1\.26 or newer which we don’t have\)

QUESTIONS?

Author: {% link "Michal" https://performance-engineering-solutions.com/team/index.html#Michal %}
