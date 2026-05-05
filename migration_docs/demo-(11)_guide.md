## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) What was in the source pipeline
Your Azure DevOps YAML defines:
- A parameter `artifactid` (default `100`)
- `pool: ubuntu-latest`
- Steps:
  - `checkout: none`
  - Echo the parameter value
  - A multi-line echo script

### 2) Dependencies / Task Groups / Variable Groups resolution
From the provided **Resolved Dependencies**:
- `task_groups`: `{}` (empty)
- `variable_groups`: `[]` (none)
- Therefore:
  - **No Task Groups/templates** existed to inline into GitHub Actions.
  - **No Variable Groups** existed to map into `${{ vars.* }}` or `${{ secrets.* }}`.
- **Task Group resolution explanation:** Not applicable—there were no task groups referenced in the source pipeline and the resolved `task_groups` object is empty.
- **Variable resolution explanation:** Not applicable—there were no variable group values provided, and the pipeline only used a parameter.

### 3) Mapping Azure concepts to GitHub Actions
- **Azure parameter** → **GitHub Actions workflow_dispatch input**
  - Azure: `parameters: artifactid`
  - GitHub: `on.workflow_dispatch.inputs.artifactid` with default `"100"`
- **Agent pool** → **runs-on**
  - Azure: `vmImage: ubuntu-latest`
  - GitHub: `runs-on: ubuntu-latest`
- **`checkout: none`**
  - GitHub Actions doesn’t have a direct `checkout: none` equivalent, so the workflow includes a step that explicitly indicates checkout is skipped.

### 4) Secrets to add to GitHub
None required.
- The source pipeline contains no secret-like values and no variable groups were provided.

### 5) How to run
- Go to the GitHub Actions tab
- Select **Demo (11)**
- Click **Run workflow**
- Optionally set `artifactid`