# Assignment 1: Multi-Branch Environment Deployment (Python - zafer)

## Short & Simple Explanation (easy)
- **Goal:** Run CI (lint + tests + build artifact) for every pull request and on pushes to `dev`/`main`.
- **Deploy:** Only deploy when pushing to branches:
  - `dev` → **QA** (deploy_qa job)
  - `main` → **Production** (deploy_prod job)
- **Artifact passing:** `ci` job creates an artifact (a `.tar.gz`) and `deploy_*` jobs download the same artifact in the same workflow run.
- **Project name:** This project is named **zafer** (see `pyproject.toml`) — matching the assignment requirement to "use CI with zafer".

## Files included (repo skeleton)
- `pyproject.toml` — project metadata (name = zafer)
- `src/app.py` — tiny Python function
- `tests/test_app.py` — unit tests (pytest)
- `requirements.txt` — test dependency
- `.github/workflows/ci-deploy.yml` — GitHub Actions workflow (CI + branch deploys)
- `.gitignore`
- `README.md` — this file

## Branch → Environment mapping (assignment)
| Branch / Event | Environment | Deploy? | Notes |
| -------------- | ----------- | ------- | ----- |
| main (push)    | production  | ✅      | Live environment |
| dev (push)     | qa          | ✅      | Test / QA environment |
| pull_request   | (none)      | ❌      | CI only (lint + test) |

## How it works (very short)
1. `ci` job runs on PRs and on pushes to `dev`/`main`. It lints, runs tests, and creates `build/output.txt` then archives it to `artifact.tar.gz` and uploads the artifact.
2. `deploy_qa` runs **only** when the event is a push to `dev`. It downloads and extracts the artifact and simulates deployment.
3. `deploy_prod` runs **only** when the event is a push to `main`. It downloads/extracts the artifact and simulates production deployment.
4. This keeps PRs non-deploying and branch pushes deploy to their mapped environments.

## Quick step-by-step (commands) — from extracted folder to GitHub
Replace `<your-username>` and `<repo-name>` appropriately.

```bash
cd zafer-python-ci               # folder from the zip

# initialize repo and push to GitHub
git init
git add .
git commit -m "Initial commit: zafer Python CI + multi-branch deploy"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main

# create and push dev branch
git checkout -b dev
git push -u origin dev
```

## How to test (recommended order)
1. **PR behavior:** Create a feature branch, change `src/app.py` or README, push and open a PR to `dev`. The Actions run should show only the `ci` job (no deploy jobs).
2. **Dev push behavior:** Merge or push commit into `dev`. The workflow run should show `ci` then `deploy_qa` (artifact used).
3. **Prod push behavior:** Merge `dev` into `main` (or push to `main`). Workflow run shows `ci` then `deploy_prod` (artifact used).

## View build output and artifacts
- Open the repository → **Actions** → select the workflow run → expand job steps to view console logs.
- To download artifacts: open the workflow run, look for **Artifacts** at the top-right and download the `build-artifact` tar.

## Notes & next steps
- Replace "Simulate deploy" steps with real deploy commands (scp/rsync/kubectl/Cloud CLI) and store credentials in repository secrets for a real pipeline.
- If you want the pipeline to show a static "zafer" user in the build output, we used the project name in `pyproject.toml`. The workflow's build output includes `Project: zafer`.
