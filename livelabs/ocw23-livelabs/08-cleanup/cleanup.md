# [Optional] Clean up your OCI environment

## Introduction

The following includes instructions for removing all resources created during the workshop. You may choose to do this immediately, or come back later after you've had more time to explore the features and capabilities covered in this lab.

Estimated time: 10 minutes

### Objectives

* Delete deployments to Kubernetes, which will also delete associated load balancers
* Delete the OKE cluster
* Clean up any remaining resources that may persist

## Task 1: Remove Kubernetes deployments

1. starting with Helm, list all installed apps:

    ```bash
    <copy>
    helm list --all-namespaces
    </copy>
    ```

    ```console
    NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    kube-prometheus-stack   monitoring      1               2023-08-17 21:24:07.462961209 +0000 UTC deployed        kube-prometheus-stack-51.0.3    v0.68.0    
    metrics-server          kube-system     1               2023-08-17 19:53:07.155075977 +0000 UTC deployed        metrics-server-3.10.0           0.6.3      
    mushop                  default         1               2023-09-15 17:42:41.980223606 +0000 UTC deployed        mushop-0.2.1                    2.0  
    </copy>
    ```

2. Repeat the following command for each app:

    ```bash
    <copy>
    helm delete -n <the corresponding namespace> <app name>
    </copy>
    ```

    For example:

    ```console
    release "kube-prometheus-stack" uninstalled
    ```

3. Delete the Neuvector deployment:

    ```bash
    <copy>
    kubectl -n keuvector delete -f ~/resources/neuvector.yaml
    </copy>
    ```

4. Delete the ingress-nginx controller:

    ```bash
    <copy>
    kubectl -n ingress-nginx delete svc ingress-nginx-controller
    </copy>
    ```

5. Delete the OKE Cluster
    1. Minimize Cloud Shell and navigate to **Kubernetes Clusters (OKE)**.
    2. Click on the Workshop cluster and then click the **`[Delete]`** button.
    3. Type in the cluster name to confirm and click **`[Delete]`** again. 
    4. It will take a few minutes to remove all the resources.

6. Navigate to **Networking** --> **Load Balancers** --> **Load balancer**. Delete any remianing load balancers in the workshop compartment.

7. Navigate to **Networking** --> **Virtual Cloud Networks**. Delete the VCNs that were created as part of the workshop (you will likely only see `oke-vcn-quick-cluster1-%%%` or something similar).

8. Navigate to **Identity & Security** --> **Domains**. Click the *Default* domain. Delete the dynamic group that you created during the Cluster Autoscaler lab.

9. Navigate to **Identity & Security** --> **Policies**. Locate and delete the policy that you created during the Cluster Autoscaler lab.


You've come to the end of the workshop. **Congrats!!**


## Acknowledgements

* **Author** - Eli Schilling - Developer Advocate
* **Contributors** - Chip Hwang, Jeevan Joseph
* **Last Updated By/Date** - August 2023
