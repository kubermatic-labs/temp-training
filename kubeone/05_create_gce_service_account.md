
## GCE Service Account

Create a service account `k1-service-account` for your Google Cloud resources in the [google cloud shell](https://ssh.cloud.google.com/cloudshell/editor) or with your local [`gcloud`](https://cloud.google.com/sdk/install) CLI.
```bash
# create new service account
gcloud iam service-accounts create k1-service-account

# get your service account id
gcloud iam service-accounts list
# get your project id
gcloud projects list

# set variables
export YOUR_PROJECT_ID="<YOUR-PROJECT-ID-HERE>"

# Use the output of the command `gcloud iam service-accounts list`.
export SERVICE_ACCOUNT="<FULL-EMAIL-FROM-OUTPUT>"


# create policy binding
gcloud projects add-iam-policy-binding $YOUR_PROJECT_ID --member "serviceAccount:$SERVICE_ACCOUNT" --role='roles/compute.admin'
gcloud projects add-iam-policy-binding $YOUR_PROJECT_ID --member "serviceAccount:$SERVICE_ACCOUNT" --role='roles/iam.serviceAccountUser'
gcloud projects add-iam-policy-binding $YOUR_PROJECT_ID --member "serviceAccount:$SERVICE_ACCOUNT" --role='roles/viewer'

# create a new json key for your service account
gcloud iam service-accounts keys create --iam-account $SERVICE_ACCOUNT k8c-cluster-provisioner-sa-key.json
```
Verify at https://console.cloud.google.com/iam-admin/serviceaccounts that the service account have been created. Now export your GCP `credentials.json` content with **`cat`**:
```bash
export GOOGLE_CREDENTIALS=$(cat ./k8c-cluster-provisioner-sa-key.json)1
```
Test if your environment variable contains the json key
```bash
echo $GOOGLE_CREDENTIALS
{ "type": "service_account", "project_id": "YUOUR PROJECT", "private_key_id": "..." }
```
