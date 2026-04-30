## Migration Guide: Azure DevOps Pipeline → GitHub Actions

### 1) What was migrated
Your Azure DevOps pipeline (single Windows job) contained these steps in order:

1. **PowerShell Script**: downloads/uses `dotnet-install.ps1` to install **.NET SDK 2.0.0** (x64) into `C:\Program Files\dotnet\`
2. **Restore**: `dotnet restore` for `$(Parameters.RestoreBuildProjects)` with detailed verbosity
3. **Build**: `dotnet build` with `--configuration $(BuildConfiguration)`
4. **Test**: `dotnet test` for `$(Parameters.TestProjects)` with `--configuration $(BuildConfiguration)`
5. **Publish**: `dotnet publish` to `$(build.artifactstagingdirectory)`
6. **Copy arm templates**: copy `Environment/**` into `$(build.artifactstagingdirectory)\Environment`
7. **Publish Artifact**: publish `$(build.artifactstagingdirectory)` as artifact named `drop` and also to a UNC path in Azure DevOps

The GitHub Actions workflow reproduces steps **1–6** and replaces **7** with GitHub’s artifact upload.

---

### 2) Dependencies / Task Groups / Variable Groups resolution
**Resolved Dependencies provided:**  
- `task_groups`: `{}` (empty)  
- `variable_groups`: `[]` (empty)

**Impact on migration:**
- There were **no task groups/templates** referenced by ID in the provided pipeline JSON that we could inline (so no template expansion was possible/needed).
- There were **no variable group values** provided, so any Azure DevOps variables/parameters that appear in the JSON (like `$(Parameters.RestoreBuildProjects)`, `$(Parameters.TestProjects)`, `$(BuildConfiguration)`) could not be resolved to concrete values from your context.

**What I did instead:**
- Mapped `$(BuildConfiguration)` → `env.BUILD_CONFIGURATION` set to `Release`.
- Added placeholders for:
  - `RESTORE_BUILD_PROJECTS` (defaulted to `**/*.csproj`)
  - `TEST_PROJECTS` (defaulted to `**/*Tests*.csproj`)
  
You should update these env values to match your actual Azure DevOps pipeline parameters.

---

### 3) Secrets / Variables mapping
Because **no variable groups** (and no secret values) were provided, there are **no `${{ secrets.* }}`** references in the workflow.

However, your Azure DevOps pipeline had a UNC publish target:
- `TargetPath: \\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)`

GitHub Actions cannot write to that UNC path unless you:
- provide network access (runner connectivity),
- and provide credentials (typically via secrets).

**Manual secrets to add (only if you need the UNC copy behavior):**
- `UNC_USERNAME`
- `UNC_PASSWORD`
(or a single credential/token approach you use for SMB access)

If you want, I can extend the workflow to copy `artifactstaging` to that UNC path using those secrets.

---

### 4) Notes about .NET SDK 2.0.0 on GitHub runners
The original pipeline installs **.NET Core SDK 2.0.0**. GitHub-hosted Windows runners may not reliably support such an old SDK (TLS endpoints, compatibility, or install script behavior).

- If your app can be built with a newer SDK, it’s strongly recommended to update the install version.
- If you must keep 2.0.0, the workflow includes the install step exactly as requested, but you may need to adjust if the install fails.

---

### 5) Artifact behavior differences
- Azure DevOps: publishes artifact to both **artifact store** and a **UNC share**.
- GitHub Actions: publishes to the **GitHub Actions artifact store** via `actions/upload-artifact`.

If you need the UNC share copy, tell me and I’ll add an SMB copy step.

---

### 6) Required follow-ups (you must confirm)
1. What are the real values for:
   - `Parameters.RestoreBuildProjects`
   - `Parameters.TestProjects`
2. What is the real `BuildConfiguration` (Release/Debug)?
3. Do you need the UNC publish target (`\\my\share\...`) in addition to GitHub artifacts?
4. Confirm whether **.NET SDK 2.0.0** is truly required.

Once you provide (1) and (3), I can produce a final “drop-in” workflow with correct project globs and optional UNC publishing.