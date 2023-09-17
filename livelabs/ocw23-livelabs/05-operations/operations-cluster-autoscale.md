# Cluster Administration and Operations

## Introduction

In the previous lab you enabled your application pods to automatically scale out when the existing capacity becomes insufficient. What happens when additional pods are required but the underlying compute capacity is not sufficient? Enter cluster auto scaling!

In this workshop you will enable the ability for the number of worker nodes in the node pool to increase automatically when additional pods need to be scheduled. With this configuration, the number of worker nodes can also decrease automatically when the extra capacity is no longer required.

<details><summary><b>Additional information: Kubernetes Cluster Autoscaler</b></summary>

You can use the Kubernetes Cluster Autoscaler to automatically resize a cluster's managed node pools based on application workload demands. By automatically resizing a cluster's node pools, you can ensure application availability and optimize costs.

The Kubernetes Cluster Autoscaler is a standalone program that:

* Adds worker nodes to a node pool when a pod cannot be scheduled in the cluster because of insufficient resource constraints.
* Removes worker nodes from a node pool when the nodes have been underutilized for an extended time, and when pods can be placed on other existing nodes.

The Kubernetes Cluster Autoscaler increases or decreases the size of a node pool automatically based on resource requests, rather than on resource utilization of nodes in the node pool.

The Kubernetes Cluster Autoscaler works on a per-node pool basis. You use a configuration file to specify which node pools to target for expansion and contraction, the minimum and maximum sizes for each node pool, and how you want the autoscaling to take place. Node pools not referenced in the configuration file are not managed by the Kubernetes Cluster Autoscaler.

To enable the Kubernetes Cluster Autoscaler to automatically resize a cluster's node pools based on application workload demands, always include resource request limits in pod specifications (requests: under resources:).

</details>

Estimated time: 20 minutes

### Objectives

* Create requisite IAM resources to allow worker nodes to manage node pools
* Deploy the auto scaler application to your Kubernetes cluster
* Force a scaling event by scheduling more pods than the cluster can place

## Task 1: Create IAM Resources
A dynamic group is required to identify the compute resources that will be granted the authorization to perform node pool scaling events.

1. Minimize Cloud Shell (or navigate the following path but *open in new window*) and navigate to **`Identity & Security`** --> **`Domains`**. While the list says *"No Items Found"* note the **Default (Current domain)** at the bottom of the list.  Click there.

    ![Default domain](images/iam-default-domain.png)

2. Navigate to **`Dynamic groups`** in the left nav menu, and **`[Create dynamic group]`**.

3. Provide a *name (i.e. oke-cluster-autoscaler-dyn-grp)* and *description*. Then enter the following rule. Be sure to input the Compartment OCID you created earlier.

    ```
    <copy>
    ALL {instance.compartment.id = 'your compartment ocid here within the single quotes'}
    </copy>
    ```

4. Create the group.

5. Now navigate to **`Identity & Security`** and under **`Identity`**, click **`Policies`**.

6. Click **`[Create policy]`**. Enter a name and description once again. I used *ocw23-oke-cluster-autoscaler-dyn-grp-policy* for the name.

7. In the **Policy Builder** section click *Show manual editor* to bring up the free-form entry field.

    ![Manual editor](images/iam-manual-policy.png)

8. Paste in the following policy statements (be sure to imput your own values for dynamnic-group-name and compartment-name):

    ```
    <copy>
    Allow dynamic-group <dynamic-group-name> to manage cluster-node-pools in compartment <compartment-name>
    Allow dynamic-group <dynamic-group-name> to manage instance-family in compartment <compartment-name>
    Allow dynamic-group <dynamic-group-name> to use subnets in compartment <compartment-name>
    Allow dynamic-group <dynamic-group-name> to read virtual-network-family in compartment <compartment-name>
    Allow dynamic-group <dynamic-group-name> to use vnics in compartment <compartment-name>
    Allow dynamic-group <dynamic-group-name> to inspect compartments in compartment <compartment-name>
    </copy>
    ```

    The final policy should look something like this:

    ![Policy example](images/iam-policy-statement.png)

    <details><summary><b>What permissions did we just create?</b></summary>
    The policy statements above grant the worker nodes (those in the compartment we specified in the dynamic group) access to manage the node pool itself (in order to change the number of worker nodes). It also grants access to create the compute resources in the compartment. **Use subnets** and **Use vnics** allows for the creation of a virtual network interface for the Compute instance, in our OKE node pool subnet.
    ---
    </details>

9. Click `[Create]` to save the new policy.

## Task 2: Apply Cluster configuration

1. Return to Code Editor and create a new file called `cluster-autoscaler.yaml`, then paste the following content:

    ```
    <copy>
    ---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "patch", "update"]
  - apiGroups: [""]
    resources:
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create","list","watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - image: iad.ocir.io/oracle/oci-cluster-autoscaler:1.27.2-9
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 300Mi
            requests:
              cpu: 100m
              memory: 300Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=oci
            - --max-node-provision-time=25m
            - --nodes=1:5:{{ node pool ocid }}
            - --scale-down-delay-after-add=5m
            - --scale-down-unneeded-time=5m
            - --unremovable-node-recheck-timeout=5m
            - --balance-similar-node-groups
            - --balancing-ignore-label=displayName
            - --balancing-ignore-label=hostname
            - --balancing-ignore-label=internal_addr
            - --balancing-ignore-label=oci.oraclecloud.com/fault-domain
          imagePullPolicy: "Always"
          env:
          - name: OKE_USE_INSTANCE_PRINCIPAL
            value: "true"
          - name: OCI_SDK_APPEND_USER_AGENT
            value: "oci-oke-cluster-autoscaler"
    </copy>
    ```

2. Make note of the command section towards the end of the above YAML content. There are a few items that will need to be adjusted:

    * `--nodes=1:5:{{ node pool ocid}}`: Here you will need to locate the OCID for your OKE node pool, and paste it.  Replace the curly braces in the process so you're left with something like: `--nodes=1:5:ocid1.nodepool.oc1.phx.aaaaaaaaptvbivi24yn7fcfgpq4jovgdgoqakwvmiaje25rp2nncj4gzflja` - This is telling the cluster autoscaler we can scale between 1 and 5 nodes. For the workshop, that's going to be fine.
    
    * The value of `--scale-down-delay-after-add=5m` was reduced for this workshop. To avoid **thrash** (unnecessarily scaling out - in - out - in) this would generally be a bit higher. 

    * The value of `--scale-down-unneeded-time=5m` was also reduced.  This is the duration that the cluster auto scaler will wait before removing resources, once the scale in threshold has been met.



You may now **proceed to the next lab**.

## Learn More

* [Oracle Docs - Cluster Auto Scaling](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengusingclusterautoscaler.htm)
* [GitHub - Kubernetes Cluster Auto Scaling](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)

## Acknowledgements

* **Author** - 
* **Contributors** -
* **Last Updated By/Date** - 