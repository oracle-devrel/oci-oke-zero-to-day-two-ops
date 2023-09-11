# Cluster Administration and Operations

## Introduction

In the previous lab you enabled your application pods to automatically scale out when the existing capacity becomes insufficient. What happens when additional pods are required but the underlying compute capacity is not sufficient? Enter cluster auto scaling!

In this workshop you will enable the ability for the number of worker nodes in the node pool to increase automatically when additional pods need to be scheduled. With this configuration, the number of worker nodes can also decrease automatically when the extra capacity is no longer required.

Estimated time: 20 minutes

### Objectives

* Create requisite IAM resources to allow worker nodes to manage node pools
* Deploy the auto scaler application to your Kubernetes cluster
* Force a scaling event by scheduling more pods than the cluster can place

## Task 1: Create IAM Resources
A dynamic group is required to identify the compute resources that will be granted the authorization to perform node pool scaling events.

You may now **proceed to the next lab**.

## Learn More

* [Oracle Docs - Cluster Auto Scaling](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengusingclusterautoscaler.htm)
* [GitHub - Kubernetes Cluster Auto Scaling](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

## Acknowledgements

* **Author** - 
* **Contributors** -
* **Last Updated By/Date** - 