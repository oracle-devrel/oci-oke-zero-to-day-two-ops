# Install monitoring with Prometheus

## Introduction

This lab will illustrate the capabilities of utilizing Prometheus to capture metrics within your Kubernetes cluster, and Grafana to view and analyize those metrics.

**Estimated module duration** 20 mins.

### Objectives

Install Prometheus and Grafana, then interact with the Grafana dashboard to view metrics within your OKE Cluster.

### Prerequisites

## Task 1: Explaining Monitoring in Kubernetes

Monitoring a service in Kubernetes involves three components

**A. Generating the monitoring data**
This is mostly done by the service itself, the metrics capability we created when building the Helidon labs are an example of this.

Core Kubernetes services may also provide the ability to generate data, for example the Kubernetes DNS service can report on how many lookups it's performed.

**B. Capturing the data**
Just because the data is available doesn't mean we can analyse it, something needs to extract it from the services, and store it. 

**C. Processing and Visualizing the data**
Once you have the data you need to be able to process it to visualize it and also report any alerts of problems.


## Task 2: Preparing for installing Prometheus

For this lab we are going to do some very simple monitoring, using the metrics in our microservices and the standard capabilities in the Kubernetes core services to generate data.  Then we'll use the Prometheus to extract the data and Grafana to display it.

These tools are of course not the only ones, but they are very widely used, and are available as Open Source projects.

### Task 2a: Configuring Helm


## End of the module, what's next ?

You can move on to the `Visualizing with Grafana` module which builds on this module, or chose from the various Kubernetes optional module sets.

## Acknowledgements

* **Author** - Jeevan Joseph, Sr. Principal Product Manager
* **Contributor** - Eli Schilling, Developer Advocate
* **Last Updated By** - Eli Schilling, August 2023