## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) Dependency Analysis (Task Groups / Variable Groups)
- **Task Groups:** `task_groups: {}` (empty)
  - **Result:** There were **no task groups/templates** referenced in the provided Azure pipeline steps, so **no inlining was required**.
- **Variable Groups:** `variable_groups: []` (none provided)
  - **Result:** There are **no variable group values** to map into `vars.*` or `secrets.*`.

**How Task Groups were resolved:**  
- Not applicable—no task group references existed in the source pipeline, and the resolved context contains no task group definitions.

### 2) Trigger Mapping
- Azure DevOps:
  - `trigger: - main`
- GitHub Actions:
  - `on: push: branches: [ main ]`

### 3) Agent / Runner Mapping
- Azure DevOps:
  - `pool: vmImage: 'ubuntu-latest'`
- GitHub Actions:
  - `runs-on: ubuntu-latest`

### 4) Checkout + Push Back to Repo
Azure DevOps used:
- `persistCredentials: true` to allow `git push`.

GitHub Actions equivalent:
- `actions/checkout@v4` with:
  - `persist-credentials: true`
  - `fetch-depth: 0` (safer for pushes/refs)

Also required:
- Job permission:
  - `permissions: contents: write`

### 5) Secrets / Vars Mapping
- The provided pipeline does **not** reference any variable groups or secrets.
- The Git identity is hard-coded in the script (`Pipeline Bot`, `pipelinebot@example.com`), so no `vars.*` or `secrets.*` mapping was necessary.

**Manual steps / secrets to add to GitHub:**
- **None required** for this specific pipeline as written.
  - It uses the default GitHub token via `actions/checkout` credentials and the workflow `contents: write` permission.

### 6) Notes on Idempotency
The Azure pipeline would fail if there’s nothing new to commit. The GitHub version includes a small guard:
- If `git diff --cached --quiet` indicates no staged changes, it exits cleanly.