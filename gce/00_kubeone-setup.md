# KubeOne Master setup

Setup a Kubeone HA cluster, like it is described in [../kubeone/README_gce.md](../kubeone/README_gce.md):
 
We will now look into the following steps:
- [../kubeone/00_install_needed_tools.md](../kubeone/00_install_needed_tools.md)

Steps to create a KubeOne Cluster on GCP :
- [../kubeone/gce/05_create_gce_service_account.md](../kubeone/gce/05_create_gce_service_account.md)
- [../kubeone/gce/10_inital_terraform_kubeone_cluster_setup.md](../kubeone/gce/10_inital_terraform_kubeone_cluster_setup.md)  
- [../kubeone/gce/20_setup_cluster_ha_master.md](../kubeone/gce/20_setup_cluster_ha_master.md)   
- [../kubeone/gce/30_setup_ha_worker.md](../kubeone/gce/30_setup_ha_worker.md)   

## (If needed) cleanup namespaces

Please cleanup deployments, ingress, and certmanager:
```bash
### if applied at the kubeone tutorial
#Delete the CustomResourceDefinitions and cert-manager itself
kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v0.15.1/cert-manager.yaml
### delete maybe some left over validation webhooks
kubectl delete -A ValidatingWebhookConfiguration --all

kubectl get ns
### delete every namespace, besides of default,kube-*
kubectl delete ns app
kubectl delete ns app-ext
kubectl delete ns cert-manager
kubectl delete ns ingress-nginx
```
**check if everything is cleaned up!**
```
kubectl get ns
NAME              STATUS   AGE
default           Active   7h43m
kube-node-lease   Active   7h43m
kube-public       Active   7h43m
kube-system       Active   7h43m
```
Check also if some old CRDs are still in place:
```
kubectl api-resources | grep cert
```
**only** `certificates.k8s.io` should be in the output
```
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest***NOTE:*** if a namespace is in state pending for longer then a few minutes, you could use the cleanup script `kubermatic/helper-scripts/kill-kube-ns.sh`
```

## (If needed) cleanup DNS Entries

Delete entries `*.DNS_ZONE.loodse.training.` at [Google Cloud DNS](https://console.cloud.google.com/net-services/dns/zones/)


## Scale your cluster (can also be done later on demand)

Scale your MachineDeployment to one MachineDeployments per AZ, see `## Add additional Machine Pools for a multi AZ cluster`:

Then modify them the type of the machines to `n1-standard-2` and `1` node per AZ.

1. Option: take a look at the script [helper-scripts/machinedeployment-patch.gce.sh](../helper-scripts/machinedeployment-patch.gce.sh). This is a very basic example that shows how easy you could automate machine deployment handling:
```bash
./helper-scripts/machinedeployment-patch.gce.sh
```

2. Option: Manual steps:
```bash
kubectl scale md -n kube-system --replicas=1 --all
```
Then edit the MachineDeployment's machine type to `n1-standard-2`:
```
kubectl edit md -n kube-system k1-pool-az-a
kubectl edit md -n kube-system k1-pool-az-b
kubectl edit md -n kube-system k1-pool-az-c
```
```yaml
      providerSpec:
        value:
          cloudProvider: gce
          cloudProviderSpec:
            assignPublicIPAddress: true
            diskSize: 50
            diskType: pd-ssd
            labels:
              k1-workers: pool1
            machineType: n1-standard-2
            # preemptible: true
```
Check if the rolling update is triggered (no need to wait until all new nodes are ready):
```bash
watch kubectl get machinedeployments.cluster.k8s.io,machinesets,machine,nodes -A
```
