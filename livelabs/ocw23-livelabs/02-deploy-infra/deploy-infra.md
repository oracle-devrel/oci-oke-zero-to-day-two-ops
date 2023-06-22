# Provision the OKE cluster and Virtual Cloud Network

## Introduction

In this lab you will levege **Quick create** feature to deploy a new OKE cluster along with all requisite virtual network resources. A couple of clicks is all it takes!

Estimated time: 30 minutes

### Objectives

Learn how to quickly deploy a new OKE cluster without much hassle, so you can focus more on deploying and managing your applications.

## Task 1: Provision an OKE Cluster

1. Navigate to **`Developer Services`** -> **`Kubernetes Clusters (OKE)`**

2. Click **`[Create cluster]`**, choose **Quick create**, and click **`[Submit]`**.

    ![quick create](images/oke-quick-create.png)

3. Provide a name for your cluster (no spaces).

4. Assign the following settings:
    1. Kubernetes API Endpoing: *Public endpoing*
    2. Node Type: *Managed*
    3. Kubernetes worker nodes: *Private workers*
    4. Shape and image:
        1. Pod shape: *VM.Standard.E3.Flex*
        2. Nmber of OCPUs: *1*
        3. Amount of memory (GB): *16*
    5. Image: *Oracle Linux 8* (latest)
    6. Node count: *2*
        *You'll add more later*

5. Click **`[Next]`**

6. Confirm the resources to be created and click **`[Create cluster]`**

7. The cluster and associated network resources will begin creating. You can close the dialog page.

8. Feel free to explore the UI a bit, the cluster will take a short while to create.  Do not proceed until the large hexagon turns green.

## Task 2: Retrieve Kubeconfig and set up some shortcuts

1. Click **`Access Cluster`** and then *copy* the command found in the second step.

    ![create kubeconfig](images/create-kubeconfig.png)

2. Open Cloud Shell

    ![cloud shell](images/cloud-shell.png)

3. Paste and run the command. This will return the message: *New config written to the Kubeconfig file in `<your home directory>/.kube/config`

4. And finally...let's make life a little easier by reducing the amount of typing required.  If you'd like, go ahead and:

    1. Create an alias for the **`kubectl`** command.

        ```<copy>alias k=kubectl</copy>```

    2. Create an alias for **`kubectl get`**

        ```<copy>alias kg='kubectl get'</copy>```

    3. Create an alias for **`kubectl delete`** (in this case we use *rm* to emulate the bash shell)

        ```<copy>alias krm='kubectl delete'</copy>```

**NOTE:** During the remainder of the workshop, `kubectl` commands will be typed out in full. If you opted to create aliases, don't forget to use them.

## Task 3: Get to know your OKE cluster

Now that Kubernetes is up and running, interacting with the cluster is pretty much the same as if you were running Kubernetes on your own equipment. Let's take a look at some of the resources that get created as part of the initial setup.

1. First and foremost, what do we know about our new OKE cluster? Type `kubectl cluster-info` to find out!

2. Take a look at the default set of namespaces with `kubectl get namespaces`. You should see *default*,*kube-node-lease*,*kube-public*, and *kube-system*.

3. Check to see which pods are running across all namespaces with `kubectl get pods -o wide -A`. You should see the likes of coredns, flannel, kube-proxy, and more.

4. Are there any ingress resources defined? Try `kubectl get ingress -A` - the results should be empty as we've not yet deployed an ingress controller or defined any ingress resources.  


You may now **proceed to the next lab**.

## Learn More

* [Oracle Container Engine for Kubernetes (OKE)](https://www.oracle.com/cloud/cloud-native/container-engine-kubernetes/)


## Acknowledgements

* **Author** - 
* **Contributors** -
* **Last Updated By/Date** -