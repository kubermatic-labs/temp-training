# Kubermatic storage class `kubermatic-fast`

The storage class `kubermatic-fast` is needed so as to cater for the creation of persistent volume claims (PVCs) for some of the components of Kubermatic. The following components need a persistent storage class assigned:

* User cluster ETCD statefulset
* Prometheus and Alertmanager (monitoring)
* Elasticsearch (logging)

**It’s highly recommended to use SSD-based volumes, as etcd is very sensitive to slow disk I/O. If your cluster already provides a default SSD-based storage class, you can simply copy and re-create it as `kubermatic-fast.`**

## Create storage class GCE

Create a `kubermatic-fast` storage class: see [`./manifests/gce.sc.kubermatic.fast.yaml`](./manifests/gce.sc.kubermatic.fast.yaml) and create a copy to your `kubermatic-setup-files` folder:
```bash
cd YOUR_TRAINING_FOLDER/kubermatic
cp ./gce/manifests/gce.sc.kubermatic.fast.yaml ./kubermatic-setup-files/
```

After everything looks fine, apply the new storage class:
```bash 
kubectl apply -f kubermatic-setup-files/gce.sc.kubermatic.fast.yaml
```
Check that you now have a new storage class installed:
```bash
kubectl get sc
```
```
NAME                        PROVISIONER            AGE
kubermatic-fast             kubernetes.io/gce-pd   12s
```
For more details about the storage class parameters of the Google CCM, see [Kubernetes Storage Class - GCE PD](https://kubernetes.io/docs/concepts/storage/storage-classes/#gce-pd).

