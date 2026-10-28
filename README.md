# 🔄 Automated CI/CD Pipeline for Cloudera Machine Learning (CML)

Automated Continuous Integration and Continuous Deployment (CI/CD) workflow using **GitHub Actions**. Automatically runs unit tests 🧪, builds an immutable Docker image 🐳, pushes to Docker Hub, and updates a live model endpoint on **CML** via the `cmlapi` Python SDK 🚀.

---

## 🏗️ Architecture Overview

```text
[ Push to main branch ] 🎯
       │
       ▼
1. Continuous Integration (`ci-build-test`) 🧪
   ├── Checkout code 📥
   ├── Setup Python 🐍
   ├── Run unit tests (`pytest`) 🧪
   └── Build & Push Docker image tagged with Git SHA (`github.sha`) 🐳
       │
       ▼ (Runs ONLY if CI passes 🛑)
2. Continuous Deployment (`cd-deploy`) 🚀
   ├── Install `cmlapi` library 📦
   ├── Authenticate via CML User API Key 🔑
   └── Trigger CML Model Deployment with new image tag 🌐

```

---

## 🔑 Required GitHub Secrets

Configure these environment variables in your repository under **Settings > Secrets and variables > Actions > New repository secret**:

| Secret Name 🔐 | Description 📋 | Example / Source |
| --- | --- | --- |
| `DOCKER_USERNAME` 🐳 | Your Docker Hub account username | `dev_user` |
| `DOCKER_PASSWORD` 🔑 | Docker Hub Personal Access Token (PAT) | `dckr_pat_xxxx...` |
| `CML_HOST` 🌐 | Base URL of your CML Workspace | `https://cml.company.com` |
| `CML_API_KEY` 🔑 | Your personal CML API access key | CML User Settings |
| `CML_PROJECT_ID` 📁 | Target CML Project UUID | Project URL identifier |
| `CML_MODEL_ID` 🎯 | Target CML Model Endpoint ID | Model Settings page |

---

## 🛠️ Workflow Configuration (`.github/workflows/ci-cd.yml`)

Create a file named `.github/workflows/ci-cd.yml` in your repository with the following code:

```
yaml
name: Production CI/CD Pipeline 🔄

on:
  push:
    branches: [ "main" ] 🎯

env:
  REGISTRY: docker.io 🌐
  IMAGE_NAME: ${{ secrets.DOCKER_USERNAME }}/cml-churn-app 📦

jobs:
  # Stage 1: Continuous Integration 🧪
  ci-build-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code 📥
        uses: actions/checkout@v4

      - name: Set up Python 🐍
        uses: actions/setup-python@v5
        with:
          python-version: "3.10"

      - name: Install Dependencies & Run Tests 🧪
        run: |
          pip install -r requirements.txt
          pytest tests/

      - name: Log in to Docker Registry 🔑
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build & Push Immutable Docker Image 🐳
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  # Stage 2: Continuous Deployment 🚀
  cd-deploy:
    needs: ci-build-test 🛑 # Hard gate: deployment waits for CI to pass
    runs-on: ubuntu-latest
    steps:
      - name: Set up Python 🐍
        uses: actions/setup-python@v5
        with:
          python-version: "3.10"

      - name: Deploy Tested Artifact via CML API v2 🌐
        env:
          CML_HOST: ${{ secrets.CML_HOST }}
          CML_API_KEY: ${{ secrets.CML_API_KEY }}
          PROJECT_ID: ${{ secrets.CML_PROJECT_ID }}
          MODEL_ID: ${{ secrets.CML_MODEL_ID }}
        run: |
          pip install cmlapi
          python -c "
          import os, cmlapi

          client = cmlapi.default_client(
              url=os.environ['CML_HOST'],
              cml_api_key=os.environ['CML_API_KEY']
          )

          deployment_body = cmlapi.CreateModelDeploymentRequest(
              cpu=1,
              memory=4,
              environment={'IMAGE_TAG': '${{ github.sha }}'}
          )

          response = client.create_model_deployment(
              project_id=os.environ['PROJECT_ID'],
              model_id=os.environ['MODEL_ID'],
              body=deployment_body
          )
          print(f'Deployment triggered! Build ID: {response.id}')
          "

```
