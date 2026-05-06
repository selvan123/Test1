## Migration Guide: Bitbucket Server → GitHub Actions (MavenHelloWorld)

### 1) Trigger mapping
- **Bitbucket Server**: `trigger: - master`
- **GitHub Actions**:  
  ```yaml
  on:
    push:
      branches: [ master ]
  ```
This runs the workflow on every push to `master`.

### 2) Agent / runner mapping
- **Bitbucket Server**: `pool: vmImage: ubuntu-latest`
- **GitHub Actions**: `runs-on: ubuntu-latest`

### 3) Maven build step mapping
- **Bitbucket** task:
  - `Maven@4`
  - `mavenPomFile: pom.xml`
  - `mavenOptions: -Xmx3072m`
  - `jdkVersionOption: 1.11`
  - `goals: package`
  - `publishJUnitResults: true` (and `testResultsFiles`)
- **GitHub Actions** equivalent:
  - Use `actions/setup-java@v4` with **Java 11**
  - Run Maven directly:
    ```bash
    mvn -Xmx3072m -f pom.xml package
    ```

> Note on JUnit publishing: the provided Bitbucket pipeline enables JUnit result publishing, but the resolved dependencies context you provided does not include any Task Group/template that would define how to publish test results in Bitbucket. In the GitHub workflow above, I kept the behavior minimal and focused on the build + WAR artifact upload. If you want, I can add a `actions/upload-artifact` for `**/surefire-reports/` and/or a test-results publishing step.

### 4) Artifact staging + publishing mapping
- **Bitbucket**:
  - `CopyFiles@2` copies `**/*.war` into `$(build.artifactstagingdirectory)`
  - `PublishBuildArtifacts@1` publishes artifact named `warfile`
- **GitHub Actions**:
  - Create a staging directory (`artifactstaging`)
  - Copy `**/*.war` into it
  - Upload with `actions/upload-artifact@v4` using `name: warfile`

### 5) Dependency/Task Group resolution (per your instructions)
- The source pipeline contains only direct tasks:
  - `Maven@4`
  - `CopyFiles@2`
  - `PublishBuildArtifacts@1`
- There are **no Task Groups / metaTasks** referenced in the provided Bitbucket YAML, so **no inlining of templates was required**.
- The provided “Resolved Dependencies” JSON includes pipeline/repo metadata only (e.g., `pipeline_name`, `project_key`, repo slug/name, default branch, `is_release`), but **no variable groups or task group definitions** were included. Therefore:
  - No variables could be replaced with `${{ vars.* }}` or `${{ secrets.* }}`.
  - No secrets are required for this pipeline as written (it only builds and uploads artifacts).

### 6) Secrets to add to GitHub
- **None required** for the pipeline as provided.
- If your real build later needs credentials (e.g., Maven deploy to Nexus/Artifactory, private dependencies), then you would add those as GitHub Secrets and wire them into Maven settings.