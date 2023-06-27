---
title: Reproducible CPU Utilization in Kubernetes/GKE
date: 2023-06-09 14:39:51
author: Michal
photos: 
 - /img/posts/cpu.png
tags: [cpu, kubernetes, monitoring]
categories:
 - [cpu]
 - [kubernetes]
 - [monitoring]
---

With the advent of containerized technologies such as Kubernetes it has become increasingly challenging to properly measure the application CPU utilization for performance testing and benchmarking purposes. In this article we will examine how to improve CPU utilization reproducibility between performance tests and therefore make our performance testing and benchmarking more reliable.

<!--more-->

# CPU Utilization in GKE - Issues

Usually when we run performance tests, we want to monitor and measure basic resource utilization metrics including the CPU utilization.
This is important because an increase in the CPU utilization usually indicates some code or configuration changes that decrease the overall performance of our application/system.

In GKE/Kubernetes we found out the default K8s monitoring provided CPU utilization is not stable and changes a lot during the test and between the tests even though the workload, application version and configuration are identical.

Below we show the CPU utilization on the pod as well as on the Kubernetes nodes during the execution of several tests with identical configuration and application version. The workload is constant during these tests except from some short ramp up period at the beginning of each test.

One test takes approximately 1h of time and it is clear that the CPU utilization is changing a lot between the tests and even somewhat during the execution of each test.


![img.01](reproducibility0.png)

![img.02](reproducibility1.png)

Our goal is to somehow stabilize the CPU utilization so that it could be used more reliably as a performance metric and for comparison purposes between performance tests.

We will explore the possibilities provided by Kubernetes.

# Review Default Kubernetes Limits and Requests Settings for our App

Our test app has the following resource limits settings:

{% codeblock %}
limits:
  cpu: "11"
  ephemeral-storage: 6Gi
  memory: 6Gi
{% endcodeblock %}

The resource requests settings for the same app are provided below:

{% codeblock %}
requests:
  cpu: 2500m
  ephemeral-storage: 1Gi
  memory: 6Gi
{% endcodeblock %}

On the surface everything looks fine, but we will learn in the next sections that the settings above can have an important impact on the CPU utilization stability.

# Kubernetes QoS classes

Kubernetes introduces 3 QoS classes: 

- Best Effort
- Burstable
- Guaranteed

By default if limits are not equal to requests Kubernetes is running the pod in a Burstable class, which may affect the CPU stability depending on the cluster load etc.

Documentation: [https://kubernetes\.io/docs/tasks/administer\-cluster/cpu\-management\-policies/\#static\-policy](https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/#static-policy)

# Requirements for the Guaranteed QoS Class

It is clear that we want to have our application pod in a Guaranteed QoS class. This way we can improve the stability of CPU utilization a lot and make our performance testing more reproducible.

Below is an example configuration of limits and requests to get the Guaranteed QoS class.

It is important to observe that limits and requests has to be configured the same way and the number of cores must be an integer and cannot be a fraction.

![img.03](reproducibility3.png)

# Adjusting Kubernetes Limits and Requests for our App

Limits and requests settings were made equal for our app and cpu was set to an integer value!

Our application resource limits:

{% codeblock %}
limits:
  cpu: "4"
  ephemeral-storage: 6Gi
  memory: 6Gi
{% endcodeblock %}

Our application resource requests:

{% codeblock %}
requests:
  cpu: "4"
  ephemeral-storage: 6Gi
  memory: 6Gi
{% endcodeblock %}

# Checking QoS class

After applying the changes to our deployment we can check the Kubernetes QoS class for our app with kubectl describe pod command and grepping for QoS, example below: 

```console
$ kubectl describe pod app-green-8546ccd7c7-9p6dk -n ns | grep QoS

QoS Class:                    __Guaranteed__
```

# Rerunning the Series of Tests

We finally have decent CPU reproducibility between runs (1.11-1.14 core on average for 8 consecutive runs)

![img.04](reproducibility2.png)

# Conclusions and Beyond

Setting the Kubernetes limits and requests have crucial impact on your CPU Utilization reproducibility/stability.

QoS classes for pods seem to be a crucial concept.

This and much more info in documentation: [https://kubernetes\.io/docs/tasks/administer\-cluster/cpu\-management\-policies/\#static\-policy](https://kubernetes.io/docs/tasks/administer-cluster/cpu-management-policies/#static-policy)

There is also one more Kubernetes concept you may want to evaluate yourself called CPU static policy, which requires Kubernetes version 1.26 or newer which we currently do not have.

Author: {% link "Michal" https://performance-engineering-solutions.com/team/index.html#Michal %}
