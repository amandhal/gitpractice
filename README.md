# CI/CD Fundamentals - GitHub Actions

A structured overview of Continuous Integration and Continuous Deployment
concepts, explored through GitHub Actions.

---

## Table of Contents

- [Task 1: Risks of Manual Deployment](#task-1-risks-of-manual-deployment)
- [Task 2: GitHub Actions Exploration](#task-2-github-actions-exploration)
- [Task 3: CI/CD Pipeline Flow](#task-3-cicd-pipeline-flow)

---

## Task 1: Risks of Manual Deployment

Manual deployment processes introduce several critical risks that can
impact software reliability and team productivity.

| Risk | Description |
|------|-------------|
| 🧑‍💻 **Human Error** | Wrong commands, wrong configs, or missed steps during deployment |
| 🖥️ **Inconsistent Deployments** | The classic *"works on my machine"* problem due to environment differences |
| 🐢 **Slow Releases** | Manual steps take time, causing delays in shipping features or critical fixes |
| ⏪ **No Easy Rollback** | Without automation, reverting a broken deployment is slow and risky |

> **Key Takeaway:** Manual deployments are error-prone, slow, and hard
> to undo — automation through CI/CD pipelines directly solves these issues.

---

## Task 2: GitHub Actions Exploration

**Repository Explored:**
[`clouddrove/github-shared-workflows`](https://github.com/clouddrove/github-shared-workflows.git)

**Workflow File:** `ci.yml`

---

### 2.1 What Triggered the Workflow?

The workflow is triggered by three events:

- **Push** to `main` or `master` branch
- **Pull Request** targeting `main` or `master` branch
- **Manual Trigger** via `workflow_dispatch`

---

### 2.2 What Jobs Are Running?

The pipeline runs the following jobs:

| Job | Purpose |
|-----|---------|
| `validate-yaml` | Validates all workflow YAML files for correct syntax |
| `lint-yaml` | Enforces consistent YAML formatting and style |
| `validate-workflows` | Ensures all workflows have required fields |
| `security-scan` | Checks for security vulnerabilities and hardcoded secrets |
| `validate-docs` | Verifies documentation exists and links are valid |
| `validate-naming` | Ensures workflows follow naming standards |
| `actionlint` | Advanced GitHub Actions workflow validation |
| `generate-docs` | Generates a Documentation Index |
| `check-deprecated` | Warns about outdated action versions |
| `validate-permissions` | Reviews workflow permissions for security |
| `ci-summary` | Aggregates results, creates a summary report, and displays statistics |

---

### 2.3 What Is the Workflow Trying to Achieve?

This CI/CD pipeline ensures all workflows in the repository are:

- ✅ **Validated** — Correct syntax and required fields are present
- 🔒 **Secure** — No hardcoded secrets or insecure permissions
- 📄 **Well-Documented** — Docs exist, links are valid, and an index is generated
- 🏷️ **Consistently Named** — Naming standards are enforced across all workflows
- 🔄 **Up to Date** — Deprecated actions are flagged for replacement

---

## Task 3: Correct CI/CD Pipeline Flow

A standard CI/CD pipeline follows these five sequential stages:

```mermaid
flowchart LR
    A[1️⃣ Write Code] --> B[2️⃣ Build Tests]
    B --> C[3️⃣ Run Tests]
    C --> D[4️⃣ Deploy Application]
    D --> E[5️⃣ Monitor Application]
