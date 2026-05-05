## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) Pipeline triggers
- **Azure DevOps**: `trigger: - main`
- **GitHub Actions**: `on: push: branches: [ main ]`

### 2) Job `A`
- **Azure DevOps**:
  - `pool: vmImage: ubuntu-latest`
  - step: `bash: echo "ABCD"`
- **GitHub Actions**:
  - `runs-on: ubuntu-latest`
  - `run: echo "ABCD"`

### 3) Job `B` and `pool: server`
- **Azure DevOps**: `pool: server` means the job runs without a hosted agent.
- **GitHub Actions**: every job must run on a runner, so the closest equivalent is `runs-on: ubuntu-latest`.
- If you truly need “no runner” behavior, you’d typically replace the step with an external service call from a lightweight runner or use a GitHub-hosted integration approach.

### 4) Task Groups (Templates) resolution
- **Resolved dependencies provided**: `task_groups: {}` (empty)
- Therefore:
  - No metaTask/task-group templates were referenced in the source pipeline.
  - No inlining was required.

### 5) Variable/secret resolution
- **Resolved dependencies provided**:
  - `variable_groups: []` (none)
- In the source pipeline, the Splunk task uses:
  - `splunk-events-api: 'SplunkConn'`
- Since `SplunkConn` looks like a connection/API credential, it should be treated as a **secret** in GitHub.

#### Secrets to add in GitHub
Add the following GitHub secret:
- `SplunkConn` → value of the Azure DevOps service connection / Splunk Events API token/credential

Mapped in workflow as:
- `splunk-events-api: ${{ secrets.SplunkConn }}`

### 6) Splunk task mapping note
The Azure DevOps task:
- `task: Splunk Observability Cloud Events@0`

In GitHub Actions, Splunk provides actions, but the exact `uses:` and input parameter names can vary by action version. In the workflow above, I used a Splunk GitHub Action placeholder:
- `uses: Splunk/splunk-otel-collector-github-action@v1`

If your repository already uses a specific Splunk GitHub Action (or you have a preferred one), replace the `uses:` line and adjust `with:` keys to match that action’s documented inputs.

### 7) What’s missing / assumptions
- No variable group values were provided (none exist in the resolved dependencies).
- No task group definitions were provided (none exist).
- The only credential-like value is `SplunkConn`, assumed to be a secret.

If you share the exact Azure DevOps Splunk task documentation (or the service connection type/token format), I can produce a more precise `uses:` mapping for the Splunk GitHub Action.