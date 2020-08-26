# Install Dependencies

Kubermatic ships with a number of Helm charts that need to be installed into the master or seed clusters. These are built so they can be configured using a single, shared `values.yaml` file. The required charts are:

* **Master cluster:** cert-manager, nginx-ingress-controller, oauth(, iap)
* **Seed cluster:** minio, s3-exporter

There are additional charts for the [monitoring](https://github.com/kubermatic/kubermatic/tree/master/charts/monitoring) and [logging stack](https://github.com/kubermatic/kubermatic/tree/master/charts/logging) which will be discussed in their dedicated chapters, as they are not strictly required for running Kubermatic.

In addition to the `values.yaml` for configuring the charts, a number of options will later be made inside a special
`KubermaticConfiguration` resource.

A minimal configuration for Helm charts sets the options. A template what you will fill out step-by-step is placed at [kubermatic-setup-files.template/values.yaml](kubermatic-setup-files.template/values.yaml). To configure your own version, please create first a copy to your `kubermatic-setup-files` folder:
```bash
cd YOUR_TRAINING_FOLDER/kubermatic                                                       
cp kubermatic-setup-files.template/values.yaml ./kubermatic-setup-files/
```

For the purpose of this document, we only need to configure a few things in the `values.yaml`, check:
```bash
grep --line-number TODO kubermatic-setup-files/values.yaml
kubermatic-setup-files/values.yaml:10:#          "auth": "TODO ADD PULL SECRET",
kubermatic-setup-files/values.yaml:20:    host: "kubermatic.TODO-STUDENT-DNS.loodse.training"
kubermatic-setup-files/values.yaml:30:    - https://kubermatic.TODO-STUDENT-DNS.loodse.training
kubermatic-setup-files/values.yaml:32:    - https://kubermatic.TODO-STUDENT-DNS.loodse.training/projects
kubermatic-setup-files/values.yaml:35:    - email: "TODO-STUDENT-EMAIL@loodse.training"
```

## Configure an image pull secret for Kubermatic images

To enable the Kubermatic master cluster to download the protected Kubermatic images, we need to configure a secret in the `values.yaml`. This will be used later for the deployment of the Helm files.
```bash
cat secrets/kubermatic-*.json
```
```json
{
  "auths": {
    "quay.io": {
      "auth": "xxx-YOUR-SECRET-KEY-xxx",
      "email": ""
    }
  }
}
```
Now replace the path `TODO ADD PULL SECRET` in the `values.yaml`:
```bash
vim kubermatic-setup-files/values.yaml
```
and then replace
```yaml
  # insert the Docker authentication JSON provided by Kubermatic here
  imagePullSecret: |
    {
      "auths": {
        "quay.io": {
          "auth": "xxx-YOUR-SECRET-KEY-xxx",
          "email": ""
        }
      }
    }
```

## Configure DEX authentication
To start simple we already added a basic configuration for a static OAuth ID.

```bash
vim kubermatic-setup-files/values.yaml
```

There you need to replace `TODO-STUDENT-EMAIL@loodse.training` with your `student-XX-xxx@loodse.training` email. The default password is `password`

```yaml
  # For testing purposes, we configure a single static user/password combination.
  staticPasswords:
    - email: "TODO-STUDENT-EMAIL@loodse.training" <<<< CHANGE
      # bcrypt hash of the string "password", can be created using recent versions of htpasswd:
      # `htpasswd -bnBC 10 "" PASSWORD_HERE | tr -d ':\n' | sed 's/$2y/$2a/'`
      hash: "$2a$10$2b2cU8CPhOTaGrs1HRQuAueS7JTT5ZHsHSzYiFPm1leZck7Mc8T4W"
      # these are used within Kubermatic to identify the user
      username: "admin"
      userID: "08a8684b-db88-4b73-90a9-3cd1661f5466"
```
**NOTE:** This is **not recommended for production!**
In a later chapter we will change the ID to a proper OIDC configuration.

## Configure target DNS

As a next step we need to set the target DNS name.

* Base URL will be: **kubermatic.*STUDENT_DNS_NAME*.loodse.training** 

  For this demonstration we will be using **kubermatic.student-00.loodse.training**

  This domain name will point to the Load balancer IP address of the Nginx ingress controller service.

  For the system services like Prometheus or Grafana, you will also want to create a wildcard DNS A record `*.kubermatic.student-00.loodse.training` pointing to the same Load Balancer IP of the Nginx ingress controller.
  
### Replace TODO-STUDENT-DNS
Next we want to configure the DNS names in the `values.yaml`. Before we create the DNS entries with the according LoadBalancer IP, we need to set the necessary domain names in the values.yaml. Cert-manager will use these domains to request the necessary certificates from Let's encrypt later on:

```bash
## replace every entry of: TODO-STUDENT-DNS
grep TODO-STUDENT-DNS kubermatic-setup-files/values.yaml

# get gcloud DNS_ZONE
gcloud dns managed-zones list
NAME                DNS_NAME                             DESCRIPTION  VISIBILITY
student-XX-training student-XX-training.loodse.training. k8c          public

## adjust to your zone name
export DNS_ZONE=student-XX-training
# sed -i 's/original/new/g' file
sed -i 's/TODO-STUDENT-DNS/'"$DNS_ZONE"'/g' kubermatic-setup-files/values.yaml

## check results
grep TODO-STUDENT-DNS kubermatic-setup-files/values.yaml
grep $DNS_ZONE kubermatic-setup-files/values.yaml
```
**Check if everything is correct and is matching your configured target DNS Zone!**
```
    host: "kubermatic.student-00.loodse.training"
    - https://kubermatic.student-00.loodse.training
    - https://kubermatic.student-00.loodse.training/projects
    - email: "student-00@loodse.training"
      username: "student-00@loodse.training"
```

## Install Dependencies

With the configuration prepared, it's now time to install the required Helm charts into the master cluster. Take note of where you placed your `values.yaml` and then run the following commands in your shell:

### Ingress `nginx-ingress-controller`
```bash
helm upgrade --install --create-namespace --wait --values ./kubermatic-setup-files/values.yaml --namespace nginx-ingress-controller nginx-ingress-controller kubermatic-repo/charts/nginx-ingress-controller/
```

**HINT:** If you want to check first how the manifests will get rendered, add `--dry-run` flag:
```bash
helm upgrade --install --create-namespace --wait --values ./kubermatic-setup-files/values.yaml --namespace nginx-ingress-controller nginx-ingress-controller kubermatic-repo/charts/nginx-ingress-controller/ --dry-run
```

After a few seconds everything should be created and you will see:
```
Release "nginx-ingress-controller" does not exist. Installing it now.
NAME: nginx-ingress-controller
LAST DEPLOYED: Mon Jun 29 21:58:45 2020
NAMESPACE: nginx-ingress-controller
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

Now check if everything is running, and you have an active service:
```bash
kubectl get pod,svc,ep -n nginx-ingress-controller 
```
```
NAME                                           READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-75f566958-2txkg   1/1     Running   0          7m18s
pod/nginx-ingress-controller-75f566958-wr4j4   1/1     Running   0          7m18s
pod/nginx-ingress-controller-75f566958-z7s9n   1/1     Running   0          7m18s

NAME                               TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
service/nginx-ingress-controller   LoadBalancer   10.103.3.87   34.91.40.238   80:31542/TCP,443:31806/TCP   7m18s

NAME                                 ENDPOINTS                                               AGE
endpoints/nginx-ingress-controller   10.244.3.2:80,10.244.4.9:80,10.244.5.2:80 + 3 more...   7m18s
```
**NOTE:** Not all cloud providers provide support for LoadBalancers. In these environments the `nginx-ingress-controller` chart can be configured to use a NodePort Service instead, which would open ports 80 and 443 on every node of the cluster. Refer to the `charts/nginx-ingress-controller/values.yaml` for more information.

### Let's Encrypt certificate manager `cert-manager`
Next install the cert-manager to get valid SSL certificates from [Let's Encrypt](https://letsencrypt.org/) by using the Kubernetes project [Cert Manager](https://cert-manager.io/docs/):

```bash
# deploy CRDs first (not managed by helm)
kubectl apply -f kubermatic-repo/charts/cert-manager/crd
# helm install/update
helm upgrade --install --create-namespace --wait --values ./kubermatic-setup-files/values.yaml --namespace cert-manager cert-manager kubermatic-repo/charts/cert-manager/
```
Now check if everything is running and you have an active service:
```bash
kubectl get pod,svc,ep -n cert-manager
```
Also let's see if a cluster issuer have been created:  
```bash
kubectl get clusterissuers.cert-manager.io 
```
```
NAME                  READY   AGE
letsencrypt-prod      True    58m
letsencrypt-staging   True    58m
```

### Dex OAuth proxy `oauth`

To place Kubermatic behind a single-sign-on (SSO) provider, we will deploy [Dex](https://github.com/dexidp/dex/blob/master/Documentation/kubernetes.md) as the next component:
```bash
helm upgrade --install --create-namespace --wait --values ./kubermatic-setup-files/values.yaml --namespace oauth oauth kubermatic-repo/charts/oauth/
```
**NOTE:** As an alternative an existing Keycloak installation could also be configured. Have a look in the [Kubermatic Docs](https://docs.kubermatic.com/kubermatic/master/advanced/oidc_config/) for more information.

Validate that the resources have been created:
```bash
kubectl -n oauth get pod,svc,ep
```
```
NAME                            READY   STATUS    RESTARTS   AGE
pod/cm-acme-http-solver-wnc6n   1/1     Running   0          43m
pod/dex-55596f57bd-fbhrh        1/1     Running   0          43m
pod/dex-55596f57bd-sxlj8        1/1     Running   0          43m

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/cm-acme-http-solver-dgn4z   NodePort    10.109.192.167   <none>        8089:31448/TCP   43m
service/dex                         ClusterIP   10.96.214.126    <none>        5556/TCP         43m

NAME                                  ENDPOINTS                         AGE
endpoints/cm-acme-http-solver-dgn4z   10.244.4.16:8089                  43m
endpoints/dex                         10.244.3.7:5556,10.244.5.7:5556   43m
```
**NOTE:** the missing ingress `ADDRESS` is ok for now, because we didn't setup or DNS entry yet! 

```bash
kubectl -n oauth get ingresses
```
```
NAME                        HOSTS                                   ADDRESS   PORTS     AGE
dex                         kubermatic.student-00.loodse.training             80, 443   52s
```
