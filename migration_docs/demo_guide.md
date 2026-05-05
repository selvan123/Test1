## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) Dependency Analysis (Task Groups / Variable Groups)
From the provided **Resolved Dependencies**:
- `task_groups`: `{}` (none)
- `variable_groups`: `[]` (none)

**Impact on migration:**
- There were **no task groups/templates** to inline.
- There were **no variable groups** to map into `${{ vars.* }}` or `${{ secrets.* }}` automatically.

### 2) Task Groups (Templates) Handling
- The Azure DevOps pipeline contains a single custom task:
  - `Splunk Observability Cloud Events@0`
- It does **not** reference any meta task group/template.
- Therefore, **no task-group inlining was required**.

### 3) Variable Resolution / Secret Mapping
In the Azure DevOps step:
- `splunk-events-api: 'SplunkConn'`

Because this looks like a connection/API credential, it should be treated as a **secret** in GitHub Actions.

**Mapping applied:**
- `SplunkConn` → `${{ secrets.SplunkConn }}`

Other fields were left as literals because no variable group values were provided:
- `environment: production`
- `eventType: New Azure DevOps deployment of test-the-tools`

### 4) Manual Steps (Secrets to Add in GitHub)
Add the following secret to the GitHub repository (or organization):
- `SplunkConn` — the value that Azure DevOps used for `splunk-events-api`

### 5) Runner / Pool Behavior Notes
Azure DevOps job **B** used:
- `pool: server` (runs without an agent)

GitHub Actions does not support a true “server/no-runner” job. The closest equivalent is to run on a standard runner:
- `runs-on: ubuntu-latest`

If you later need a truly agentless approach, you’d typically replace the action with a direct API call from a lightweight runner or use GitHub-hosted alternatives.