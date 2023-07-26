# Setting up Auto scaling

## Introduction

Auto scaling is a critical part of managing Kubernetes at scale. By leveraging metrics generated within your cluster, you can define when additional resources are provisioned to support your workloads, thus reducing the need for more manual work.

**Estimated module duration** 20 mins.

### Objectives

This module explores how you can configure Kubernetes to automatically scale the number of pods for your service to handle changes in demand with no manual intervention.

### Prerequisites

You need to complete the **Horizontal Scaling** module.

## Task 1: Horizontal autoscaling - based on CPU load or Memory usage

We've seen how we can increase (or decrease) the number of pods underlying a service and that the service will automatically balance the load across the pods for us as the pod count changes.

This is great, but it required manual intervention to change the number of pods, and that means we need to keep an eye on what's happening. Fortunately for us Kubernetes has support for automating this process based on rules we define.

This is the simplest form of auto scaling, though it is also the least flexible as CPU and memory usage may not always be the most effective indicator of when scaling is required. 

To gather the data we need to have installed the metrics server. This is a replacement for an older Kubernetes service called heapster. 

Note that it's also possible to use Prometheus as a data source which will allow you to auto-scale on custom metrics, for example the number of requests to your service. 

For now we are going to use the simplest approach of the metrics server.

### Task 1a: Installing the metrics server

  1. Install a helm repo that contains the chart for the metrics server
  
    ```bash
    <copy>
    helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
    </copy>
    ```
  
  2. Update the repo
  
    ```bash
    <copy>
    helm repo update
    </copy>
    ```
  
    ```
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "metrics-server" chart repository
    ```

  3. In the OCI Cloud Shell install the metrics server by typing
  
    ```bash
    <copy>
    helm -n kube-system install metrics-server metrics-server/metrics-server --version 3.10.0
    </copy>
    ```

    ```
    NAME: metrics-server
    LAST DEPLOYED: Fri Jun  23 19:11:45 2023
    NAMESPACE: kube-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    ***********************************************************************
    * Metrics Server                                                      *
    ***********************************************************************
      Chart version: 3.10.0
      App version:   0.6.3
      Image tag:     registry.k8s.io/metrics-server/metrics-server:v0.6.3
    ***********************************************************************
    ```

It will take a short time for the metrics server to start up, but you can check the progress using kubectl

4. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl get deployments -n kube-system</copy>
    ```
  
    ```
    NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
    coredns                3/3     1            1           17m
    kube-dns-autoscaler    1/1     1            1           17m
    kubernetes-dashboard   1/1     1            1           15m
    metrics-server         1/1     1            1           39s
    ```

### Task 2: Using the captured metrics

Once the metrics server is running (it will have an AVAILABLE count of 1) you can get information on the state of the system

  1. Let's look at how the nodes in the cluster are doing. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl top nodes</copy>
    ```

    ```
    NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
    10.0.10.2   73m          3%     1561Mi          23%       
    10.0.10.3   109m         5%     1834Mi          27%       
    10.0.10.4   157m         7%     1232Mi          18%       
    ```
 
    If you get an response `error: metrics not available yet` then it just means that the metrics server is running, but hasn't completed it's initial data capture yet. Wait a short while and try the `kubectl top nodes` again.
  
    Note that this was from a cluster with three nodes, depending on the size of your cluster you will see a different number of nodes output.
  
  2. We can also see the status of the pods in terms of what they are using. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl top pods</copy>
    ```

    ```
    NAME                           CPU(cores)   MEMORY(bytes)   
    mushop-api-88bd7c499-qcwcp           1m           12Mi            
    mushop-assets-7f97df7bbd-zsmgc       1m           7Mi             
    mushop-edge-5d6f7ccf67-xdvr2         1m           10Mi            
    mushop-session-864cb5b4db-vqpxr      1m           7Mi             
    mushop-storefront-7b685595df-sczzf   1m           1Mi               
    ```

    **Note** that like the other times we've used kubectl this uses the namespace configured as the default when you ran the create-namespace.sh command

  3. Let's have a look at what's happening in the kube-system namespace. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl top pods -n kube-system</copy>
    ```

    ```
    NAME                                    CPU(cores)   MEMORY(bytes)   
    coredns-85f945b85d-c7ddm                4m           10Mi            
    coredns-85f945b85d-vrbgp                3m           10Mi            
    coredns-85f945b85d-zjzgl                4m           10Mi            
    kube-dns-autoscaler-7bcf86584b-8tct6    1m           6Mi             
    kube-flannel-ds-5w4k8                   3m           11Mi            
    kube-flannel-ds-b7dwm                   5m           13Mi            
    kube-flannel-ds-jhvgf                   3m           11Mi            
    kube-proxy-chk7h                        1m           12Mi            
    kube-proxy-lrlc5                        4m           11Mi            
    kube-proxy-mp784                        2m           12Mi            
    kubernetes-dashboard-77f54dc48f-9sbdh   1m           10Mi            
    metrics-server-55d8d46477-fzztx         5m           17Mi            
    proxymux-client-99q9z                   1m           8Mi             
    proxymux-client-ffprz                   1m           8Mi             
    proxymux-client-lr7ft                   1m           8Mi   
    ```

<details><summary><b>Kubernetes dashboard and the metrics-server</b></summary>


The metrics server provides information on the current use of resources in the cluster, the Kubernetes dashboard has just been updated to version 2, at some point the plan is that the dashboard will be able to connect to the metrics-server and you will be able to see the pod CPU and memory usage in the dashboard

---

</details>

You can see in the output above that all of the pods are using very small amounts of CPU here, this is because we're not really putting any load on them. Let's run a script that will put load on the services to see what's happening.

If your cloud shell session is new or has been restarted then the shell variable `$EXTERNAL_IP` may be invalid, expand this section if you think this may be the case to check and reset it if needed.

<details><summary><b>How to check if $EXTERNAL_IP is set, and re-set it if it's not</b></summary>

**To check if `$EXTERNAL_IP` is set**

If you want to check if the variable is still set type `echo $EXTERNAL_IP` if it returns the IP address you're ready to go, if not then you'll need to re-set it, there are a couple of ways to do this, expand the appropriate section below.

<details><summary><b>Reminder: How to get the external IP address</b></summary>

  - Open the OCI cloud shell 

  - You are going to get the value of the `EXTERNAL_IP` for your environment. This is used to identify the DNS name used by an incoming connection. In the OCI cloud shell type

  ```bash
  <copy>kubectl get svc -n ingress-nginx</copy>
  ```

  ```
  NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
  ingress-nginx-controller             LoadBalancer   10.96.182.204   130.162.40.241   80:31834/TCP,443:31118/TCP   2h
  ingress-nginx-controller-admission   ClusterIP      10.96.216.33    <none>           443/TCP                      2h
  ```

  - Look for the `ingress-nginx-controller` line and note the IP address in the `EXTERNAL-IP` column. In this case that's `130.162.40.121` but it's almost certain that the IP address you have will differ. IMPORTANT, be sure to use the IP in the `EXTERNAL-IP` column, ignore anything that looks like an IP address in any other column as those are internal to the OKE cluster and not used externally. 

  - IN the OCI CLoud shell type the following, replacing `<external ip>` with the IP address you retrieved above.
  
  ```bash
  export EXTERNAL_IP=<external ip>
  ```
  
---

</details>

</details>

  4. In Cloud Shell, create a new shell script called `loadgen.sh` and add the following code:

    ```
    #!/bin/bash -f
    SCRIPT_NAME=`basename $0`
    if [ $# -eq 0 ]
      then
        echo "Missing arguments supplied to $SCRIPT_NAME, you must provide :"
        echo " 1st arg External IP address of the ingress controller service load balancer"
        exit -1 
    fi
    EXTERNAL_IP=$1
    i=0
    while true;do
    i=$[$i+1]
    echo "Iteration $i"
    curl --silent -o /dev/null http://$EXTERNAL_IP
    # Do a little sleep so we don't totally overload the server
    sleep $2
    done
    ```

    To exit and save: *esc* + `:wq` + [Enter]
  
  5. Start a load generator. In the OCI Cloud Shell.
  
    ```bash
    <copy>bash loadgen.sh $EXTERNAL_IP 0.1</copy>
    ```

    Note that the 0.1 controls how long the script waits, depending on how fast things respond below you may need to adjust the rate up (fewer requests) or down (more requests) If you chose an especially powerful processor you may need to open another cloud window in your browser and run a second load in that as well, or adjust the CPU available to the pod.

    ```
    Iteration 1
    Iteration 2
    Iteration 3
    Iteration 4
    .
    .
    .
    ```

  6. Open up a new window on the OCI console
  
  7. Open up the OCI Cloud shell in the new window
  
    Let the script run for about 75 seconds (the iteration counter reaches over 750) This will let the load statistics level out.

    This will have increased the load, to see the increased load

  8. In the **new** OCI Cloud Shell type
  
    ```bash
    <copy>kubectl top pods</copy>
    ```
  
    ```
    NAME                           CPU(cores)   MEMORY(bytes)   
    mushop-api-88bd7c499-qcwcp           1m           12Mi            
    mushop-assets-7f97df7bbd-zsmgc       1m           7Mi             
    mushop-edge-5d6f7ccf67-xdvr2         7m           25Mi            
    mushop-session-864cb5b4db-vqpxr      1m           7Mi             
    mushop-storefront-7b685595df-sczzf   16m           1Mi              
    ```

    You'll see the CPU load has increased, as the data is averaged over a short period of time you may have to wait a short while (say 30 seconds) for it to update.

  9. In the **new** OCI Cloud Shell (after waiting for a little bit) type
  
    ```bash
    <copy>kubectl top pods</copy>
    ```
  
    ```
    NAME                           CPU(cores)   MEMORY(bytes)   
    mushop-api-88bd7c499-qcwcp           1m           12Mi            
    mushop-assets-7f97df7bbd-zsmgc       1m           7Mi             
    mushop-edge-5d6f7ccf67-xdvr2         1m           25Mi            
    mushop-session-864cb5b4db-vqpxr      1m           7Mi             
    mushop-storefront-7b685595df-sczzf   1m           1Mi           
    ```

    Notice that in this particular example the CPU here for the *nginx* is at 251m (your number may be different). This is actually the limit allowed in the storefront deployment.yaml file, which has a resource restriction of 250 milli CPU specified. 

    If the load does not reach 250 then you may need to adjust the request rate, or open another cloud shell in a new browser tab / window and run another load generator to ensure you can hit the limit (and auto scaling will kick in once we've configured it)

    ```yaml
            resources:
              limits:
                # Set this to be quarter of a CPU for now
                cpu: "250m"
    ```

We can also get the current resource level for the container using kubectl and the jsonpath capability

  10. In the OCI Cloud Shell (substitute your storefront pod name) type 
  
  ```bash
  kubectl get pod storefront-79c465dc6b-x8fbl -o=jsonpath='{.spec.containers[0].resources.limits.cpu}'
  ```
 
    ```
    250m
    ```
 
  11. In the OCI Cloud shell window running the load generator stop it using Control-C

<details><summary><b>Namespace level resource restrictions and implications</b></summary>

In Kubernetes you can place restrictions on pods as we've seen above, however it's also possible to apply a quota on the namespace. All of the pods (or other resources) in the name space have to be within that limit. To do this you create a ResourceQuota (there is one for each namespace) The ResourceQuota allows you to specify limits on CPU, memory storage and also it's possible to extend these with custom resources (e.g. GPU if your cluster hardware has these) Some resources can also be limited on the resource count e.g. number of configmaps.

If an action like adding an object would cause the limits applied to a namespace to be exceeded then the action will be rejected. This applies to auto scaling as well!

Importantly, if you have a ResourceQuota like CPU or memory usage applied to a namespace, then each pod must also have a limit applied to it, if it's not there then either a default will be applied or the create object request will be rejected. (the exact action is deployment specific)

---

</details>

## Task 3: Configuring the autoscaler

That we have hit the limit is almost certainly a problem, it's quite likely that the performance of the service is limited out because of this. Of course it may be that you have made a deliberate decision to limit the service, possibly to avoid overloading the back end database (though as it's an ATP database it can scale automatically for you if the load gets high)

To fix this problem we need to add more pods, but we don't want to do this by hand, that would mean we'd have to be monitoring the system all the time. Let's use a the Kubernetes autoscale functionality to do this for us 

Setup autoscale (normally of course this would be handled using modifications to the YAML file for the deployment)

**If your CPU load was not exceeding 125** then reduce the `--cpu-percent` setting to a lower number so that it's below the required level. For example if the CPU load for the pod was not exceeding 50 (so 20% of the allowed load on a single pod) then a setting of `--cpu-percent=15` will trigger a scaling.

  1. In the OCI Cloud Shell create an auto scaller
  
    ```bash
    <copy>kubectl autoscale deployment <tbd> --min=2 --max=5 --cpu-percent=25</copy>
    ```
  
    ```
    horizontalpodautoscaler.autoscaling/tbd autoscaled
    
    ```
    The autoscaler will attempt to achieve a target CPU load of 25%, (or whatevery you used) adding or removing pods to the deployment as needed to meet that goal, but never going below two pods or above 5


    We can see what the system has found by looking in the Horizontal Pod Autoscalers

  2. In the OCI Cloud Shell type 
  
    ```bash
    <copy>kubectl get horizontalpodautoscaler tbd</copy>
    ```

    ```
    NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
    <tbd>        Deployment/tbd           25%/25%   2         5         2          44s
    ```

    Note that because we told the auto scaler that we wanted a minimum of 2 pods the REPLICAS has already increased to 2. Because this deployment is "behind" the tbe service the service will automatically arrange for the load to be balanced across both pods for us.

    Of course typing `horizontalpodautoscaler` takes some time, so kubectl has an alias setup for us, we can just type `hpa` instead, for example

    ```
    kubectl get hpa tbd
    NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
    tbd           Deployment/tbd          25%/25%   2         5         2          56s
    ```

    A few points on the output The TARGET column tells us what the **current** load is first, this is the average across **all** the pods in the deployment, Then the target load the autoscaler will aim to achieve by adding or removing pods.

    You can get more detail on the autoscaler state

  3. Getting the autoscaler details. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl describe hpa tbd</copy>
    ```

    ```
    Name:                                                  tbd
    Namespace:                                             default
    Labels:                                                <none>
    Annotations:                                           <none>
    CreationTimestamp:                                     Fri, 23 Jun 2023 19:07:36 +0000
    Reference:                                             Deployment/tbd
    Metrics:                                               ( current / target )
      resource cpu on pods  (as a percentage of request):  25% (63m) / 25%
    Min replicas:                                          2
    Max replicas:                                          5
    Deployment pods:                                       2 current / 2 desired
    Conditions:
      Type            Status  Reason              Message
      ----            ------  ------              -------
      AbleToScale     True    ReadyForNewScale    recommended size matches current size
      ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
      ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
    Events:           <none>
    ```

    Now restart the load generator program. Note that you may need to change the sleep time from 0.1 to a different value if the load generated is not high enough to trigger an autoscale operation, but don't set it to low!

  4. In the OCI Cloud Shell where you were running the load balancer previously (substitute the IP address for the ingress for your cluster) If needed run multiple in different cloud shells.
  
    ```bash
    <copy>bash loadgen.sh $EXTERNAL_IP 0.1</copy>
    ```


    ```
    Itertation 1
    Itertation 2
    ...
    ```

  5. Switch to the second cloud shell window in your browser.

    Allow a short time for the load to be recorded, then look at the load on the pods (you may have to adjust the request frequency in the script if the load does not increase enough to trigger autoscaling

  6. Let's check there is a load. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl top pods</copy>
    ```

    ```
    NAME                           CPU(cores)   MEMORY(bytes)   
    nginx2                         45m          421Mi           
    nginx-2340cds0                 117m         288Mi           
    nginx-79c465dc6b-x8fbl         251m         1931Mi          
    ```

    Notice that the load on the pods has increased

  7. Let's look at the autoscaler. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl get horizontalpodautoscaler tbd</copy>
    ```

    ```
    NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    tbd   Deployment/tbd   43%/25%   2         5         3          16m
    ```

    The current load (in this case 73%) is above the 50% target. The autoscaler will have kicked in and be starting to do it's thing. 

  8. Let's look at the autoscaler details. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl describe hpa storefront</copy>
    ```
  
    ```
    Name:                                                  tbd
    Namespace:                                             default
    Labels:                                                <none>
    Annotations:                                           <none>
    CreationTimestamp:                                     Fri, 23 Jun 2023 19:07:36 +0000
    Reference:                                             Deployment/storefront
    Metrics:                                               ( current / target )
      resource cpu on pods  (as a percentage of request):  43% (87m) / 25%
    Min replicas:                                          2
    Max replicas:                                          5
    Deployment pods:                                       3 current / 5 desired
    Conditions:
      Type            Status  Reason              Message
      ----            ------  ------              -------
      AbleToScale     True    SucceededRescale    the HPA controller was able to update the target scale to 5
    ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
    ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
    Events:
      Type    Reason             Age   From                       Message
      ----    ------             ----  ----                       -------
      Normal  SuccessfulRescale  57s   horizontal-pod-autoscaler  New size: 3; reason: cpu resource utilization (percentage of request) above target
      Normal  SuccessfulRescale  11s   horizontal-pod-autoscaler  New size: 5; reason: cpu resource utilization (percentage of request) above target
    ```

    In fact it seems that in the time between the commands above the short term average load increased sufficiently that the autoscaler having determined it wanted three pods then updated to realize it wanted 5 pods to meet the load. In this case if we look at the pods list we can see the details there


  9. Let's look at those pods. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl top pods</copy>
    ```

    ```
    NAME                           CPU(cores)   MEMORY(bytes)   
    stockmanager-86dbc548d-b8cbl   19m          425Mi           
    storefront-79c465dc6b-2lvlr    64m          385Mi           
    storefront-79c465dc6b-br87r    82m          647Mi           
    storefront-79c465dc6b-jk5cq    73m          402Mi           
    storefront-79c465dc6b-m55tc    62m          428Mi           
    storefront-79c465dc6b-x8fbl    84m          1933Mi          

    ```

    All 5 pods are running and the service is distributing the load amongst them. Actually some of the storefront pods above are probably still in their startup phase as I gathered the above data immediately after getting the auto scale description.

  10. Let's get the autoscaler summary again. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl get hpa tbd</copy>
    ```
  
    ```
    NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
    tbd   Deployment/tbd   30%/25%   2         5         5          30m
    ```

    We can see that the average load across the deployment is still over 25%, if is had not reached the maximum number of pods then the auto scaler would be trying to meet our goal of no more than 50% average load by adding additional pods.

    If you want you can also see the pods being added in the Kubernetes dashboard,  

  11. Open the Kubernetes Dashboard

  12. Make sure you are in your namespace

  13. In the left menu Under workloads select **Deployments** then click on the `tbd` deployment

  14. In the **Pods** section you can see that in this case it's scaled to 5 pods

    ![Seeing the change in pod count in the dashboard](images/autoscaling-pods-increased.png)

  15. Scroll down to the **Events** section and you can seen the changes it's made

    ![History of pod count changes in the dashboard](images/autoscaling-dashboard-events.png)

  16. You can see the pod details by opening the replica set.

    ![Details of the auto scaled pods](images/autoscaling-pods-list.png)

  17. In the load generator window(s) stop the script by typing Control-C

    Note that the metrics server seems to operate on a decaying average basis, so stopping the load generating script will not immediately drop the per pod load. This means that it may take some time after stopping the load generator script for the autoscaler to start removing unneeded pods.   

    The autoscaler tries not to "thrash" the system by starting and stopping pods all the time. Because of this it will only remove pods every few minutes rather than immediately the load becomes low, additionally it will also only remove a few pods at a time. The [autoscaler documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details) describes the algorythm.

    For now let's delete the autoscaler to we can proceed with the next part of the lab with a known configuration

  18. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl delete hpa tbd</copy>
    ```

    ```
    horizontalpodautoscaler.autoscaling "tbd" deleted
    ```

    Note that this just stops changes to the number of pods, any existing pods will remain, even if there are more (or less) than specified in the deployment document. Of course now the auto scaler has been deleted regardless of the load that number will no longer change automatically.

    To return to the numbers of replicas originally defined we'll use kubectl

  19. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl scale --replicas=1 deployment tbd</copy>
    ```

    ```
    deployment.apps/tbd scaled
    ```

    Now let's check what's happening

  20. In the OCI Cloud Shell type
  
    ```bash
    <copy>kubectl get deployment storefront</copy>
    ```

    ```
    NAME           READY   UP-TO-DATE   AVAILABLE   AGE
    stockmanager   1/1     1            1           4d2h
    storefront     1/1     1            1           4d2h

    ```

The number of pods is now back to one (it may be that you get a report of 2 pods still running, in which case try getting the deployments again a little bit later).

## Task 4: Autoscaling on other metrics
We have here looked at how to use CPU and memory to determine when to autoscale, that may be a good solution, or it may not. Kubernetes autoscaling can support the use of other metrics to manage autoscaling.

These other metrics can be other Kuberneties metrics (known as custom metrics) for example the number of requests to the ingress controller, or (with the provision of the [Prometheus Adaptor (helm chart)](https://github.com/prometheus-community/helm-charts)) any metric that Prometheus gathers. This last is especially useful as it means you can autoscale on what are effectively business metrics.

It's also possible to autoscale on metrics provides from outside Kubernetes (these are known as external metrics). this is only recommended as a last resort however due to the security implications, and custom metrics are preferred.

The [autoscaler docs explain some of these](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)

## Task 4: Other forms of AutoScaling
Here we have looked a how to have Kubernetes create new pods based on load (horizontal auto scaling). 

OKE also supports vertical auto scaling (increasing the allowed resource limits on a pod) and to allow for situations where a pod may not sale horizontally that well. 

There is another form of auto scaling which can be enabled called cluster autoscaling, this monitors the cluster to see if it's got any pods that can't be deployed because there are no available workers with enough suitable capacity, it will create new worker nodes as needed and if there are to many worker nodes for the overall load in the cluster it will shuffle pods of nodes and stop the node.

### Additional information

Autoscaler and rolling upgrades - The autoscaler will correctly operate in conjunction with rolling upgrades.

## End of the module, what's next ?

You have reached the end of this section of the lab. The next module is `Rolling Updates`

## Acknowledgements

* **Author** - 
* **Contributor** - 
* **Last Updated By** - 