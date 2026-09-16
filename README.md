# Reuseable Workflows

Reusable GitHub Actions workflows for common CI tasks.



## Docker Build and Push to AKS via Helm 

Workflow file:

```text
.github/workflows/docker-build.yml
```

Use this when you want to build a Docker image and push it to Azure Container Registry.

It can be used for Go, Java, Python, Node and other apps as long as the project has a Dockerfile.

### Example for a Go Project

Your Go project can look like this:

```text
mygoapp
Dockerfile
go.mod
go.sum
main.go
.github
  workflows
    build.yml
```

Your Go project needs a Dockerfile.

Example:

```dockerfile
FROM golang:1.24

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -o app .

EXPOSE 8080

CMD ["./app"]
```

Then create `.github/workflows/build.yml` in the Go project.

```yaml
name: Build Go App

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Environment to deploy to"
        required: true
        type: choice
        options:
          - DEV
          - UAT
          - PROD
        default: DEV

permissions:
  id-token: write
  contents: read

jobs:
  build:
    uses: DonGranda/reuseable_workflow/.github/workflows/build-acr-aks.yml@v2.0.0
    secrets: inherit
    with:
      environment: ${{ inputs.environment }}
 
```

The reusable workflow will checkout the project, login to Azure, login to ACR, build the Docker image and push it to ACR.

The image will look like this:

```text
myacr.azurecr.io/mygoapp:25
```

The number comes from the GitHub Actions run number.

## Azure Setup

The Docker workflow uses Azure OIDC to login to Azure.

Add these secrets to the GitHub repository that is calling the reusable workflow:

```text
AZURE_CLIENT_ID
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
```

The Azure identity also needs permission to push to your Azure Container Registry.

A common role is:

```text
AcrPush
```

Your Azure App Registration also needs a federated credential for GitHub Actions OIDC.

## References 

https://www.incredibuild.com/blog/best-practices-to-create-reusable-workflows-on-github-actions

https://medium.com/@reach2shristi.81/github-actions-with-reusable-workflows-27e99afac2e2

https://medium.com/@nipun.jayasanka10/mastering-github-actions-the-power-of-reusable-workflows-for-clean-scalable-ci-cd-a56d603ca8c6
