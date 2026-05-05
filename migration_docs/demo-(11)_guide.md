## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) What was in the source pipeline
Your Azure DevOps YAML defines:
- A pipeline parameter: `artifactid` (default `100`)
- `pool: ubuntu-latest`
- Steps:
  - `checkout: none`
  - `echo "Param value ${{ parameters.artifactid }}"`
  - A multi-line echo script

### 2) Dependencies / Task Groups / Variable Groups resolution
From the provided **Resolved Dependencies**:
- `task_groups`: `{}` (empty)
- `variable_groups`: `[]` (none)
- Therefore:
  - **No Task Groups/templates existed to inline** (there were no metaTask references in the provided pipeline).
  - **No variable groups existed to map** into `${{ vars.* }}` or `${{ secrets.* }}`.

**How Task Groups were resolved:**  
- Not applicable. The source pipeline contains no task-group references, and `task_groups` is empty.

**How Variable Groups were resolved:**  
- Not applicable. `variable_groups` is empty, and the pipeline contains no variable-group variables.

### 3) Parameter mapping
Azure DevOps parameter:
- `parameters.artifactid` → GitHub Actions input:
  - `inputs.artifactid`

Implementation detail:
- GitHub Actions uses `workflow_dispatch` inputs to mimic runtime parameters.
- Default is set to `"100"` to match the Azure DevOps default.

### 4) `checkout: none` mapping
Azure DevOps step `checkout: none` means the repository is not checked out.
- GitHub Actions always starts with a clean workspace, but the `actions/checkout` step is optional.
- To mirror the intent, the workflow **does not include** `actions/checkout`.
- I added a small log step (“Checkout (disabled)”) to make the behavior explicit.

### 5) Secrets to add to GitHub
None required.
- The provided pipeline does not reference any secrets or variable-group values.
- `task_groups` and `variable_groups` are empty.

If you later add real build/deploy tasks that require credentials (e.g., registry login, cloud deploy), then you should add those as GitHub repository/environment secrets and map them to `${{ secrets.NAME }}`.