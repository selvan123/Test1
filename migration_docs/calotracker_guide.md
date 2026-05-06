## Migration Guide: Bitbucket Server Pipeline → GitHub Actions

### 1) What this workflow does (mapping from your Bitbucket pipeline)
Your Bitbucket pipeline has 4 stages. The GitHub Actions workflow replicates them in order:

1. **Checkout**
   - Bitbucket: `git branch: 'main', url: ...`
   - GitHub Actions: `actions/checkout@v4` with `ref: main`

2. **Build with Docker Compose**
   - Bitbucket:
     - `sudo docker compose build`
     - tag `calotracker-2-calotracker:latest` → `yashtnaik/calotracker:v1`
     - remove intermediate image
   - GitHub Actions: same commands in a `run:` step.

3. **Push to DockerHub**
   - Bitbucket logs in using DockerHub credentials and pushes `${DOCKER_IMAGE}:${IMAGE_TAG}`
   - GitHub Actions does the same using GitHub secrets.

4. **Update `manifest/deployment.yaml` and push back to `main`**
   - Bitbucket:
     - `sed` replaces the image tag
     - commits with `jenkins` identity
     - pushes using HTTPS credentials embedded in the URL
   - GitHub Actions: same `sed`, commit, and HTTPS push.

---

### 2) Dependency/Template resolution (Task Groups)
- **Task Groups / metaTask templates:** None were provided in the resolved dependencies context.
- Therefore, **no template inlining was required**. Every Bitbucket step was directly translated into an equivalent GitHub Actions step.

---

### 3) Variable/credential resolution
Your Bitbucket pipeline used:
- `DOCKERHUB_CREDENTIALS = credentials('DockerHub')`
- `GIT_CREDENTIALS = credentials('GitHub')`

In Jenkins, those credentials typically expose:
- `*_USR` and `*_PSW` environment variables (as your pipeline references `${DOCKERHUB_CREDENTIALS_USR}` and `${DOCKERHUB_CREDENTIALS_PSW}`).

In GitHub Actions, you must create secrets with the names used by the workflow:

#### Secrets to add in GitHub
Add these **repository secrets** (or environment secrets) in GitHub:

- `DOCKERHUB_USR`  
- `DOCKERHUB_PSW`  
- `GITHUB_USR`  
- `GITHUB_PSW`  

These correspond to:
- DockerHub username/password from Bitbucket credential **DockerHub**
- GitHub username/password from Bitbucket credential **GitHub**

> Note: GitHub now strongly recommends using a **Personal Access Token (PAT)** instead of a password for `GITHUB_PSW`.

---

### 4) Notes / recommended improvements
- The workflow includes a guard:
  - If `manifest/deployment.yaml` doesn’t change, it exits without failing.
- The workflow uses `sudo` like your Bitbucket pipeline. If your runner permissions differ, you may remove `sudo`.

---

### 5) How to run
- The workflow triggers on every push to `main`.
- It will build, push `yashtnaik/calotracker:v1`, then update `manifest/deployment.yaml` and commit/push the change back to `main`.