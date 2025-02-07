This is an example how you can push an docker image to google cloud artifacts.

```yml
name: Release
on:
  push:

jobs:
  docker-release:
    name: Tagged Docker release to Google Artifact Registry
    runs-on: ubuntu-latest

    permissions:
      contents: "read"
      id-token: "write"

    steps:
      - id: checkout
        name: Checkout
        uses: actions/checkout@v2

      - id: auth
        name: Authenticate with Google Cloud
        uses: google-github-actions/auth@v2
        with:
          token_format: access_token
          workload_identity_provider: <your-workload-identity-provider>
          service_account: <your-service-account>
          access_token_lifetime: 300s

      - name: Login to Artifact Registry
        uses: docker/login-action@v3
        with:
          registry: europe-west10-docker.pkg.dev
          username: oauth2accesstoken
          password: ${{ steps.auth.outputs.access_token }}

      - name: Get tag
        id: get-tag
        run: echo ::set-output name=short_ref::${GITHUB_REF#refs/*/}

      - id: docker-push-tagged
        name: Tag Docker image and push to Google Artifact Registry
        uses: docker/build-push-action@v6
        with:
          push: true
          tags: |
            <docker-registry>:${{ steps.get-tag.outputs.short_ref }}
            <docker-registry>:latest
```
