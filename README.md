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
