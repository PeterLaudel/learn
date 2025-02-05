# Google Cloud Artifacts with Github Actions

## Create service account

```sh
#Authorize gcloud to use your Google Cloud credentials
gcloud auth login
gcloud auth list
gcloud projects list
```

Set the project ID so we are in the write environment

```sh
gcloud config set project $MY_PROJECT_ID
# check if it's active
gcloud config get-value project
```

Create an service account for github actions

```sh
gcloud iam service-accounts create githubactions \                                                              
    --description="service acct for github actions" \                     
    --display-name="Github Actions"

gcloud iam service-accounts list
```

Enable Google's IAM API for use.

```sh
gcloud services enable iamcredentials.googleapis.com \
  --project "${PROJECT_ID}"
````

Create a workload identity pool that will manage the GitHub Action's roles in Google Cloud's permission system.

```sh
gcloud iam workload-identity-pools create "${WORKLOAD_IDENTITY_POOL}" --project="${PROJECT_ID}" --location="global" --display-name="${WORKLOAD_IDENTITY_POOL}"
```

Get the unique identifier of that pool.

```sh
gcloud iam workload-identity-pools describe "${WORKLOAD_IDENTITY_POOL}" --project="${PROJECT_ID}" --location="global" --format="value(name)"
```

Create a provider within the pool for GitHub to access.

```sh
gcloud iam workload-identity-pools providers create-oidc "${WORKLOAD_PROVIDER}" --project="${PROJECT_ID}" --location="global" --workload-identity-pool="${WORKLOAD_IDENTITY_POOL}" --display-name="${WORKLOAD_PROVIDER}" --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" --issuer-uri="https://token.actions.githubusercontent.com" --attribute-condition="attribute.repository_owner == '${ORGANIZATION}'"
```

Allow a GitHub Action based in your repository to login to the service account via the provider.

```sh


gcloud iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" --project="${PROJECT_ID}" --role="roles/iam.workloadIdentityUser" --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"
```

Ask Google to return the identifier of that provider.

```sh
gcloud iam workload-identity-pools providers describe "${WORKLOAD_PROVIDER}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="${WORKLOAD_IDENTITY_POOL}" \
  --format="value(name)"
```
That will return a string that you should save for later. We'll use it in our GitHub Action.

Finally, we need to make sure that the service account we created at the start has permission to muck around with Google Artifact Registry.


```sh
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/artifactregistry.admin"
```


Verify that it worked.

```sh
gcloud projects get-iam-policy $PROJECT_ID \
    --flatten="bindings[].members" \
    --format='table(bindings.role)' \
    --filter="bindings.members:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com"
```