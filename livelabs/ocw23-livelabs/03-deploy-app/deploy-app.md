# Deploying your first application to OKE

## Introduction

In this lab you'll deploy a few different resources and get to know how Kubernetes operates on OCI. If you're already deployed Kubernetes yourself in another environment, you'll find it functions pretty much the same...only without all the effort required to actually deploy and manage the cluster.

Estimated time: 30 minutes

### Objectives

* Deploy a sample app and expose it to the internet using an OCI Load Balancer.
* Deploy an NGINX Ingress controller.
* Deploy a second application and expose it to the internet via the Ingress controller.

## Task 1: Deploy NGINX and corresponding load balancer.

1. From Cloud Shell, run the following command:

    ```
    <copy>
    kubectl run nginx --image=nginx --ports=80
    <copy>
    ```

2. Use `kubectl get pods` to check status of the pod creation.

3. Create a service to expose the application. The cluster is integrated with OCI Cloud Conroller Manager (CCM). As a result, creating a service of type *--type=LoadBalancer* will expose the pod tot he internet using an OCI Load Balancer. Run the following command:

    ```
    <copy>
    kubectl expose pod nginx --port=80 --type=LoadBalancer
    </copy>
    ```

4. You may choose to minimize Cloud Shell and navigate to **Networking** -> **Load Balancers** in the Cloud Console to watch the resource create. Otherwise, run the following command to check status:

    ```
    <copy>
    kubectl get svc
    </copy>
    ```

5. It may take a minute or two to complete. But when it does, you'll see the public IP address of the Load Balancer. Copy that into a new browser tab to check your work.

    ![kubectl get svc](images/nginx-lb-service.png)

    ![sample app page](images/sample-app-page.png)

## Task 2: Deploy the NGINX-Ingress Controller

The `ingress-nginx` controller is built and mainained as part of the Kubernetes project. Deplying it to your cluster is as simple as running one command. After that, we'll dive into how to configure and use the controller.

1. Run the following command:

    ```
    <copy>
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml
    </copy>
    ```

    ![Create controller](images/create-nginx-controller.png)

2. You can see it creates a number of resources including a new namespace, service accounts, RBAC settings, and more.  Check the services created in the new namespace `ingress-nginx`.

    ```
    <copy>
    kubectl -n ingress-nginx get svc -o wide ingress-nginx-controller
    </copy>
    ```

3. Jot down that external IP address.  You'll likely use it quite a bit throughout the rest of the workshop.  You can also set an environment variable if you'd like (replace *`<External IP>`* with the one you just noted).

    ```
    <copy>
    export EXTERNAL_IP=<your external IP here>
    </copy>
    ```

*Notice* that it created a service type of Load Balancer with a public IP address? While its great to be able to easily expose a single deployment with a LoadBalancer, that may not give you the best control over your resource utilization.  With the ingress controller, we send all inbound traffic to a single load balancer, and tell NGINX how to distribute that traffic within our OKE cluster.

## Task 3: Deploy another app and configure ingress

    1. Create manifest files
    2. Deploy
    3. Test

## Task 4: 


This part is pretty straightforward.
You may now **proceed to the next lab**.


## Acknowledgements

* **Author** - 
* **Contributors** -
* **Last Updated By/Date** -