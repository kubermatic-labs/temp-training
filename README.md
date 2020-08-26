# Kubermatic Introduction Guide

**Introduction:** The purpose of this write-up is to document the process of setting up a Kubermatic cluster.

Kubermatic is a cluster-as-a-service solution that provides managed Kubernetes for your infrastructure.

##### Terminology

- **User/Customer cluster** – A Kubernetes cluster created and managed by Kubermatic
- **Seed cluster** – A Kubernetes cluster which is responsible for hosting the master components of a customer cluster
- **Master cluster** – A Kubernetes cluster which is responsible for storing the information about users, projects and SSH keys. It hosts the Kubermatic components and might also act as a seed cluster.
- **Seed datacenter** – A definition/reference to a seed cluster
- **Node datacenter** – A definition/reference of a datacenter/region/zone at a cloud provider (aws=zone, digitalocean=region, openstack=zone)


**N.B -- For further references, kindly check the official documentation** https://docs.kubermatic.io/

**Pre-requisite:** 
- Founded Kubernetes knowledge and tooling usage

**Deliverables:**

* Kubermatic Installation
  * Combined Master/Seed setup
  * Kubermatic UI / REST-API
  * Monitoring / Logging Stack
* User cluster creation for applied datacenters
  * GCE
  * AWS
* Introduction into main operational tasks and concepts

## REQUIREMENTS

* It is assumed that the reader is fairly familiar with how Kubernetes and Linux works and the usage of some common tooling
  * kubectl
  * vim or other CLI editor
  * helm3

* You finished KubeOne Training: [../kubeone](../kubeone) and already have a KubeOne Kubernetes cluster (GCP will be used for this exercise). 

* To understand the general prerequisites take a look at [Cluster Requirements](https://docs.kubermatic.com/kubermatic/master/requirements/cluster_requirements/)


## Setup Master Cluster
*N.B -- It is advisable to use a fresh KubeOne cluster (or you uninstall the Nginx ingress controller and cert-manager deployments if you want to use the same cluster that was created in the kubeone-deployment-guide) since following the steps in this write-up, the Kubermatic Helm chart already contains Nginx ingress and cert-manager charts. These two charts are tightly coupled together with some predefined parameters that are known to work with Kubermatic, hence it is advisable to use them (instead of the Nginx ingress controller and cert-manager installation steps that was used in the kubeone-deployment-guide).*

**Please check your environment to use:**
 - [Kubeone Cluster: `gce/00_kubeone-setup.md`](gce/00_kubeone-setup.md)
 - or [GKE Cluster: `gce/gke-cluster/00_gke-cluster-setup.md`](gce/gke-cluster/00_gke-cluster-setup.md)

## INSTALLATION

* See [../helpful_commands.md](../helpful_commands.md) section to install `kubectl` auto-completion and `kubens`/`kubectx` for fast  namespace/context switching.

* Connect to your running KubeOne or GKE Kubernetes cluster. This is the cluster we will use as the new Kubermatic master cluster:
    * KubeOne:
        ```bash
        cd K1_SETUP_FOLDER
        export KUBECONFIG=$PWD/k1-kubeconfig
        cd -
        ````
    * GKE Cluster: [`gce/gke-cluster/10_login_cluster.sh`](gce/gke-cluster/10_login_cluster.sh):
      ```bash
      gce/gke-cluster/10_login_cluster.sh
      ```

* Create now an empty folder for your kubermatic setup files:
    ```bash
    cd TRAINING_REPO
    mkdir kubermatic-setup-files
    ```
* Create a `kubermatic-fast` storage class, see [gce/10_setup_kubermatic_storage_class.md](gce/10_setup_kubermatic_storage_class.md):
  
* Install helm v3: [15_install_helm_3.md](15_install_helm_3.md)

* Install the needed dependencies (Ingress, CertManager): [20_install_dependencies.md](20_install_dependencies.md)

* Setup your need DNS records in Google Cloud DNS: [gce/21_setup_kubermatic_dns.md](gce/21_setup_kubermatic_dns.md)

* To install the Kubermatic master components, we need to setup the Kubermatic Operator next. The Operator will manage the installation of Kubermatic: [40_install_kubermatic_operator.md](40_install_kubermatic_operator.md)

* To create a new user cluster we need to set up a nested seed cluster as the last step:
  - [gce/50_add_seed_cluster.md](gce/50_add_seed_cluster.md)
  - [gce/51_setup_seed_dns.md](gce/51_setup_seed_dns.md)
  - [gce/52_create_user_cluster.md](gce/52_create_user_cluster.md)

## Explore Kubermatic Features

* Add an additional AWS datacenter to try out the multi cloud management capabilities: [aws/60_add_aws_to_datacenter.md](aws/60_add_aws_to_datacenter.md)
* Configure admin settings [65_admin_dashboard.md](65_admin_dashboard.md)
* Set presets for cloud credentials: [66_presets.md](66_presets.md)
* ... more will follow

## Next Steps

* Execute the tutorials: [Kubermatic Docs > Tutorials](https://docs.kubermatic.com/kubermatic/master/tutorials/)
* Get familiar with the advanced config possibilities: [Kubermatic Docs > Advanced](https://docs.kubermatic.com/kubermatic/master/advanced/)
* Execute some typical operational tasks: [Kubermatic Docs > Operation](https://docs.kubermatic.com/kubermatic/master/operation/)   

### Kubermatic Installer Script
To deploy the whole stack in a more convenient way, you can use the script [`helper-scripts/kubermatic-deploy.sh`](./helper-scripts/kubermatic-deploy.sh).
With this the different parts of the stack can be deployed in a fast way. This is helpful for deploying a new Kubermatic release and deploying changes due to upgrades or config changes in an easy way:

```bash
./helper-scripts/kubermatic-deploy.sh
```
```
Usage: kubermatic-deploy.sh" (master|seed) path/to/VALUES_AND_CONFIG_FILES path/to/CHART_FOLDER (monitoring|logging|kubermatic|kubermatic-deployment-only)
```
Combine different stack deployments
```bash
### Deploys everything BESIDES monitoring and logging
./helper-scripts/kubermatic-deploy.sh master ./kubermatic-setup-files kubermatic-repo/charts kubermatic

### Deploy Logging
./helper-scripts/kubermatic-deploy.sh master ./kubermatic-setup-files kubermatic-repo/charts logging

### Deploy Monitoring
./helper-scripts/kubermatic-deploy.sh master ./kubermatic-setup-files kubermatic-repo/charts kubermatic-deployment-only monitoring

### Update/Install only kubermatic components (UI, API Server, Controllers)
./helper-scripts/kubermatic-deploy.sh master ./kubermatic-setup-files kubermatic-repo/charts kubermatic-deployment-only
```

**NOTE: Our development team is currently working on improving the Kubermatic Operator that will replace more and more parts of the helm based installation. Stay tuned and take a look at our docs from time to time to get the latest info: https://docs.kubermatic.com/kubermatic/master/changelog/**
