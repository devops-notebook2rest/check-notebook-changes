# check-notebook-changes
A github action that checks a Jupyter notebook repository for changes. If there are no changes, it will exit and thus cancel the pipeline at that point.
# 📘 poll-notebook-repo

[![GitHub Actions](https://img.shields.io/badge/GitHub%20Action-v1-blue)](https://github.com/devops-notebook2rest/check-notebook-changes)

**check-notebook-changes** is a GitHub Action that checks whether a notebook repository has new commits since the last run. Use it to **conditionally trigger downstream workflows** only when there are changes in your notebook repo.

---

## ⚡ Features

- Polls any Git repository for the latest commit SHA.
- Uses GitHub Actions cache to detect new commits.
- Outputs `should_run` for conditional execution in downstream workflows.
- Modular and reusable for Notebook2REST pipelines or other automation.

---

## 🛠️ Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `notebook_repository_url` | The URL of the notebook repository to poll (e.g., `https://github.com/org/notebook.git`) | ✅ |

---

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `should_run` | `true` if a new commit was detected and downstream jobs should run, `false` otherwise |
| `latest_commit` | The latest commit SHA of the repository (useful for tagging Docker images, logging, etc.) |

---

## 🔧 Usage Example

```yaml
jobs:
  check:
    runs-on: ubuntu-latest
    outputs:
      should_run: ${{ steps.poll.outputs.should_run }}
    steps:
      - name: Poll notebook repository
        id: poll
        uses: your-org/poll-notebook-repo@v1
        with:
          notebook_repository_url: https://github.com/devops-notebook2rest/vl-laserfarm.git

  build_and_deploy:
    needs: check
    if: needs.check.outputs.should_run == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Notebook2REST pipeline
        uses: your-org/notebook2rest-deploy@v1
        with:
          notebook_repository_url: https://github.com/devops-notebook2rest/vl-laserfarm.git
          docker_registry: ${{ env.DOCKER_REGISTRY }}
          image_name: ${{ env.IMAGE_NAME }}
          aws_region: ${{ env.AWS_REGION }}
          eks_cluster_name: ${{ env.EKS_CLUSTER_NAME }}
