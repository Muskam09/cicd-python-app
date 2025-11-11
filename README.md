# Advanced CI/CD Pipeline with GitHub Actions

This repository demonstrates a sophisticated, multi-stage CI/CD pipeline built entirely with **GitHub Actions**. The primary focus is not on the application code itself, but on the implementation of modern DevOps automation practices for testing, building, and deploying applications securely and efficiently.

## 🎯 Project Goal

The purpose of this project is to showcase advanced GitHub Actions features to build a robust, production-ready workflow. This includes:

* **Multi-Platform Testing:** Using `matrix` strategy to validate code across different operating systems and Python versions.
* **Docker Integration:** Automatically building and publishing container images to **DockerHub**.
* **Environment Management:** Using GitHub Environments (`development`, `staging`) with required manual approvals.
* **Workflow Safety:** Implementing branch protection rules, mandatory status checks, and concurrency management.
* **Manual Control:** Enabling manual workflow runs (`workflow_dispatch`) with inputs to select specific artifacts for deployment.

## ⚙️ Technologies Used

* **CI/CD:** GitHub Actions
* **Containerization:** Docker, DockerHub
* **Testing:** Python (Unit Tests)
* **Code:** YAML (for workflow definitions)

## 🏗️ Workflow Architecture

The pipeline is defined in `.github/workflows/` and is designed to handle both Pull Requests and pushes to the `main` branch with distinct logic.

### 1. CI (Pull Request Trigger)

Triggered on every push to a Pull Request:
1.  **Run Matrix Tests:** A `matrix` strategy spins up four separate jobs to test against:
    * `ubuntu-latest` + `python 3.8`
    * `ubuntu-latest` + `python 3.9`
    * `windows-latest` + `python 3.8`
    * `windows-latest` + `python 3.9`
2.  **Status Check:** The success or failure of this CI job is enforced as a **mandatory status check** in the `main` branch protection rule, blocking any PR from being merged if tests fail.

### 2. CD (Main Branch & Manual Trigger)

Triggered either by a `workflow_dispatch` (manual run) or a merge to `main`:
1.  **Build & Push Docker:** After tests pass, a job logs into DockerHub (using repository secrets) and builds/pushes the application's Docker image.
2.  **Deploy to Development:** Automatically deploys the new image to the `development` environment.
3.  **Manual Approval:** The workflow **pauses** and waits for a **manual approval** before promoting the build to the next stage.
4.  **Deploy to Staging:** After approval, the *exact same build* is deployed to the `staging` environment.

## ✨ Key Features Implemented

This workflow showcases several critical DevOps patterns:

* **Matrix Testing Strategy:**
    Ensures cross-platform compatibility by running tests in parallel on different OS and language versions.
    ```yaml
    strategy:
      matrix:
        python-version: ["3.8", "3.9"]
        os: [ubuntu-latest, windows-latest]
    ```

* **Manual Deployment with Inputs (`workflow_dispatch`):**
    Allows a user to manually trigger the workflow and *choose which artifact to deploy*, providing flexibility outside the standard PR flow.
    ```yaml
    on:
      workflow_dispatch:
        inputs:
          artifact-to-deploy:
            description: 'Which artifact to deploy (e.g., windows-3.8)'
            required: true
            type: choice
            options:
            - windows-3.8
            - ubuntu-3.9
    ```

* **GitHub Environments & Manual Approval:**
    Uses GitHub Environments to protect `staging`. A deployment to this environment requires a manual review and click from an authorized approver.
    ```yaml
    jobs:
      deploy-to-staging:
        environment:
          name: staging
          url: [https://staging.example.com](https://staging.example.com)
        # ... steps
    ```

* **Concurrency Management:**
    Prevents multiple, outdated workflows from running simultaneously. New runs on the same PR will automatically cancel any previous, in-progress runs.
    ```yaml
    concurrency:
      group: ${{ github.workflow }}-${{ github.head_ref || github.run_id }}
      cancel-in-progress: true
    ```

* **Branch Protection Rules:**
    The `main` branch is protected to enforce quality and process:
    1.  **Require Pull Request:** All changes *must* come through a PR.
    2.  **Require Status Checks:** The "Python CI" matrix job *must* pass before merging.

## 🔧 Repository Configuration

To replicate this setup, the following configurations are required in the repository settings:

### 1. Repository Secrets

(Settings > Secrets and variables > Actions)
* `DOCKERHUB_USERNAME`: Your DockerHub username.
* `DOCKERHUB_TOKEN`: A DockerHub Access Token used for logging in.

### 2. GitHub Environments

(Settings > Environments)
Two environments must be created:

* **`development`:**
    * No protection rules.
* **`staging`:**
    * **Required reviewers:** Add users or teams who must approve deployments.

### 3. Environment Secrets

(Settings > Environments > [Environment Name] > Secrets)
Secrets specific to each environment (like database passwords or API keys) are stored here.

* **Secrets for `development`:**
    * `DB_PASSWORD`
    * `API_KEY`
* **Secrets for `staging`:**
    * `DB_PASSWORD`
    * `API_KEY`

### 4. Branch Protection

(Settings > Branches > Add rule)
* **Pattern:** `main`
* **Enable:** "Require a pull request before merging"
* **Enable:** "Require status checks to pass before merging"
    * Check the box for the "Python CI" job.

## 🚀 How to Trigger the Pipeline

### 1. Pull Request (CI)
1.  Create a new branch: `git checkout -b feature/my-new-feature`
2.  Make changes and commit.
3.  Push the branch: `git push origin feature/my-new-feature`
4.  Open a Pull Request on GitHub.
5.  The "Python CI" matrix test will run automatically.

### 2. Manual Run (CD)
1.  Go to the "Actions" tab in the GitHub repository.
2.  Select the "CI/CD Pipeline" workflow from the list.
3.  Click the "Run workflow" dropdown.
4.  Select the artifact you wish to deploy from the input.
5.  Click "Run workflow".
