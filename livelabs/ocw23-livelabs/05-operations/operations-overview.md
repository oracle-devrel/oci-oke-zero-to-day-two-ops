# Cluster Administration and Operations

## Introduction

In this lab you will explore some of the fundamental capabilities of Kubernetes and how to execute those common tasks within OKE. This includes:

1. Administration tasks: 
    1. Using the Kubernetes Dashboard
    2. Cluster and Node Pool Upgrades
    3. Creating and scaling node pools
    4. Leveraging Preemtible worker nodes
    5. Monitoring your cluster with OCI Logging Analytics
2. Operational tasks:
    1. Namespaces
    2. Horizontal pod scaling
    3. Perform rolling updates
    4. Rolling back an update
    5. Node Affinity (taints and tolerations)
    6. Persistent storage
    7. Working with secrets (OCI Vault and Azure Key Vault)
    8. Utilizing image pull secrets

Estimated time: 45 minutes

### Objectives

In this lab you will familiarize yourself with a variety of administration and operations activities commonly used with OKE. You may notice many of these are standard, Kubernetes functionality, there are a number of OKE-specific capabilities that make managing your Kubernetes environment a little bit easier.

## Task 1: Deploy the Kubernetes Dashboard

The Kubernetes dashboard is a managed add-on and can be deployed when the cluster is first created. For the sake of this workshop, we chose to walk through the process of enabling this add-on later.

1. Open the OCI console and navigate to **Developer Services** -> **Kubernetes Clusters (OKE)**. 

2. Locate and click the cluster you created in Lab 2.

3. Under **Resources** on the left-hand side of the screen, click *Add-ons*.

4. Click **`Manage Add-ons`** and then click the *Kubernetes dashboard* add-on. In the dialog that appears, click the checkbox to enable it. Then **`Save changes`**.

    ![enable dashboard](images/enable-k8s-dashboard.png)

5. 


## Task 2: Setting up Auto scaling


Instructions here

## Task 3: Create a logging namespace

...

## Task 4: Scaling your deployment with a new replica set

...

## Task 5: Rolling back a deployment update

...

## Task 6: Creating a persistent volume claim with Block Storage

...

## Task 7: Create new secrets

Create a secret in OCI Vault and another in Azure Key Vault (maybe).

## Task 8: Create and utilize an imagePullSecret

...


## Acknowledgements

* **Author** - 
* **Contributors** -
* **Last Updated By/Date** - 