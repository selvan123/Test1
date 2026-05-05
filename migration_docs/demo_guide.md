## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) Dependency Analysis (Task Groups / Variable Groups)
- **Task Groups**: The provided **resolved dependencies** contain `task_groups: {}` (empty).  
  - ✅ Result: There were **no task groups/templates** referenced in the source pipeline that required inlining.
- **Variable Groups**: The provided **resolved dependencies** contain `variable_groups: []` (none).  
  - ✅ Result: There were **no variable-group values** to map into `vars.*` or `secrets.*`.

### 2) Step-by-Step Mapping

#### Job A
**Azure DevOps**
```yaml
- job: A
  steps:
  - bash: echo "ABCD"
```

**GitHub Actions**
- Mapped to a standard job running on `ubuntu-latest`.
- Uses a `run:` step:
```yaml
jobs:
  A:
    runs-on: ubuntu-latest
    steps:
      - name: Echo
        run: echo "ABCD"
```

#### Job B (Splunk Observability Cloud Events)
**Azure DevOps**
```yaml
- job: B
  pool: server
  steps:
  - task: Splunk Observability Cloud Events@0
    inputs:
      splunk-events-api: 'SplunkConn'
      environment: 'production'
      eventType: 'New Azure DevOps deployment of test-the-tools'
```

**GitHub Actions**
- GitHub Actions does not have a direct equivalent to Azure DevOps `pool: server` (agentless execution).  
  - ✅ Practical approximation: run on `ubuntu-latest`.
- The Azure DevOps input `splunk-events-api: 'SplunkConn'` is treated as a **secret reference** because it looks like a connection/API credential.
  - Mapped to: `${{ secrets.SplunkConn }}`

```yaml
- name: Splunk Observability Cloud Events
  uses: splunk/splunk-observability-cloud-events-action@v0
  with:
    splunk-events-api: ${{ secrets.SplunkConn }}
    environment: production
    eventType: New Azure DevOps deployment of test-the-tools
```

### 3) Variable / Secret Resolution
Because **no variable groups** were provided, only the explicit connection-like value was mapped:

#### Secrets to add in GitHub
Add the following GitHub secret:
- `SplunkConn`  
  - Used as `${{ secrets.SplunkConn }}` for the Splunk Observability Cloud Events action.

### 4) Notes / Assumptions
- The Splunk task was mapped to a GitHub Action:
  - `splunk/splunk-observability-cloud-events-action@v0`
- If your Splunk organization uses a different action name/version, replace the `uses:` line accordingly while keeping the same `with:` inputs.