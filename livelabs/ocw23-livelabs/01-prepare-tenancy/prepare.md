# Prepare your tenancy

## Introduction

In this lab you'll provision a couple of requisite resources, familiarize yourself with some of the console features (if not already), and validate resource availability. If using a brand new tenancy, you will have sufficient capacity to complete the entire workshop.

Estimated time: 15 minutes

### Objectives

Ensure sufficient capacity is available to complete all labs and provision pre-requisite resources.


## Task 1: Validate resource quotas / availability

1. Log into your OCI tenancy (if not already) and use the hamburger menu to navigate to **`Governance & Administration`** -> **`Limits, Quotas and Usage`**
2. The initial display will show Compute service limits. Select **AD-1** for Scope, then click the **[Resource]** drop down and type *cores for standard.e3*. Select the first result. Check the **Available** column to ensure you have at least 3 cores available.

    ![compute quota](images/quota-compute.png)

3. Clear the **[Resource]** filter and type *Memory for standard.e3.flex* then select eh first result. Ensure you have at least 48GB available. (you'll allocate 16GB per node in your cluster)

## Task 2: 

In this activity you'll be creating a new IAM user with API signing key and auth token. The user will be leveraged later in the workshop to enable

2. Create IAM user
3. Generate API Signing Key
4. Generate Auth Token
5. ...don't lose items 2 and 3



You may now **proceed to the next lab**.



## Acknowledgements

* **Author** - 
* **Contributors** -
* **Last Updated By/Date** -