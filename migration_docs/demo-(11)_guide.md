## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) What was migrated
Your Azure DevOps pipeline (`Demo (11)`) contains:
- A parameter: `artifactid` (default `100`)
- `pool: ubuntu-latest`
- Steps:
  - `checkout: none`
  - `echo "Param value ${{ parameters.artifactid }}"`
  - A multi-line echo script

This maps cleanly to a single GitHub Actions workflow with one job running on `ubuntu-latest`.

---

### 2) Parameters mapping
**Azure DevOps**
- `parameters: artifactid` with default `100`

**GitHub Actions**
- Implemented as a `workflow_dispatch` input:
  - `inputs.artifactid` default `"100"`

In the step, the value is referenced as:
- `${{ inputs.artifactid }}`

---

### 3) Agent / runner mapping
**Azure DevOps**
- `pool: vmImage: ubuntu-latest`

**GitHub Actions**
- `runs-on: ubuntu-latest`

---

### 4) Checkout behavior
**Azure DevOps**
- `- checkout: none`

**GitHub Actions**
- GitHub Actions defaults to no checkout unless you add `actions/checkout`.
- To preserve intent, I added an explicit step that just echoes that checkout is skipped.

---

### 5) Task Groups (Templates) resolution
**Resolved Dependencies provided:**
- `task_groups: {}` (empty)
- No variable groups were provided (`variable_groups: []`)

✅ Result:
- There were **no task groups** referenced in the provided Azure YAML, and the context contains **no task group definitions** to inline.
- Therefore, **no template/task-group inlining was required**.

---

### 6) Variable resolution
**Resolved Dependencies provided:**
- `variable_groups: []` (none)

✅ Result:
- There were **no variable group values** to map to `${{ vars.* }}` or `${{ secrets.* }}`.
- The pipeline only used the parameter `artifactid`, so no secret/variable mapping was needed.

---

### 7) Secrets to add to GitHub
None required for this specific pipeline because:
- No variable groups were provided
- No secret-like values were present in the YAML

If you later add build/deploy steps that require credentials (e.g., package registry tokens, cloud credentials), you should add them as GitHub Secrets and reference them as:
- `${{ secrets.YOUR_SECRET_NAME }}`