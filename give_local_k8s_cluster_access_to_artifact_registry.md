# Give a local kubernetes environment access to your google artifact registry

First login to gcloud:

```sh
gcloud auth login
```

Eventually you have to give the right permission to the user:

```sh
gcloud projects add-iam-policy-binding <project-id> \\n --member="user:<email>" --role="roles/artifactregistry.reader
```

Then create a credential secret in the kubernetes clust:

```sh
kubectl create secret docker-registry gcr-credentials \
    --docker-server=<registry-uirl> \
    --docker-username=oauth2accesstoken \
    --docker-password="$(gcloud auth application-default print-access-token)" \
    --namespace argocd
```

And then add in your deployment yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: argocd
  name: {{ .Values.name }}
    ...
    spec:
      ...
      imagePullSecrets:
        - name: gcr-credentials
```
