# Reuseable Workflows

Reusable GitHub Actions workflows for common CI tasks.

## Table of Contents

1. [Docker Build and Push](#docker-build-and-push)
2. [Maven Test](#maven-test)
3. [Maven Package](#maven-package)
4. [Azure Setup](#azure-setup)

## Docker Build and Push

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
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  build:
    uses: DonGranda/reuseable_workflow/.github/workflows/docker-build.yml@v1.0.0
    with:
      acr-name: myacr
      acr-login-server: myacr.azurecr.io
      image-name: mygoapp
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

The reusable workflow will checkout the project, login to Azure, login to ACR, build the Docker image and push it to ACR.

The image will look like this:

```text
myacr.azurecr.io/mygoapp:25
```

The number comes from the GitHub Actions run number.

## Maven Test

Workflow file:

```text
.github/workflows/mvn-test.yml
```

Use this for a Java project that uses Maven.

Example:

```yaml
name: Java Tests

on:
  pull_request:

jobs:
  test:
    uses: DonGranda/reuseable_workflow/.github/workflows/mvn-test.yml@v1.0.0
    with:
      java-version: 21
      java-distro: temurin
```

The workflow will setup Java, cache Maven packages, run the tests and upload the test report.

Your project should have `pom.xml` and `mvnw`.

## Maven Package

Workflow file:

```text
.github/workflows/mvn-package.yml
```

Use this when you want to build a JAR from a Maven project.

Example:

```yaml
name: Java Package

on:
  push:
    branches: [main]

jobs:
  package:
    uses: DonGranda/reuseable_workflow/.github/workflows/mvn-package.yml@v1.0.0
    with:
      java-version: 21
      java-distro: temurin
```

The workflow will run Maven package with tests skipped and upload the JAR as an artifact.

It also returns the project version as `app_version`.

Example of using the output:

```yaml
jobs:
  package:
    uses: DonGranda/reuseable_workflow/.github/workflows/mvn-package.yml@v1.0.0
    with:
      java-version: 21
      java-distro: temurin

  show-version:
    needs: package
    runs-on: ubuntu-latest
    steps:
      - name: Show version
        run: echo "Version is ${{ needs.package.outputs.app_version }}"
```

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
