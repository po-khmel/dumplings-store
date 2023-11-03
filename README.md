# Momo Store, aka Dumpling Shop #2

<!-- <img width="900" alt="image" src="https://user-images.githubusercontent.com/9394918/167876466-2c530828-d658-4efe-9064-825626cc6db5.png"> -->

- [Momo Store, aka Dumpling Shop #2](#momo-store-aka-dumpling-shop-2)
  - [Build stage](#build-stage)
  - [Test stage](#test-stage)
  - [Release stage](#release-stage)
  - [Deploy to test VM](#deploy-to-test-vm)
  - [Infrastructure](#infrastructure)
  - [For Developers](#for-developers)
    - [Frontend](#frontend)
    - [Backend](#backend)

**DEV**: The build, static code test, release, and deployment process to the test VM is done in GitLab CI in this project.
- The root [`.gitlab-ci.yml`](./.gitlab-ci.yml) triggers downstream pipelines: for [Frontend](./frontend/.gitlab-ci.yml) and [Backend](./backend/.gitlab-ci.yml) 

**PROD**: The build, test, and release process is the same as in DEV. Deployment to "prod" in Kubernetes is described in the [DumplingInfra](https://github.com/po-khmel/dumplings-infra) project.

*All sensitive values are stored as GitLab CI variables*

## Build stage

**Multistage Dockerfiles** are used: [Frontend Dockerfile](./frontend/Dockerfile) and [Backend Dockerfile](./backend/Dockerfile).
- Container patch versions are assigned according to `CI_COMMIT_SHORT_SHA`.
- Docker containers are stored in the GitLab Container Registry.
- The `VUE_APP_API_URL` variable is set via `--build-arg`, and its value depends on the **commit message**:
  - If the commit contains "test-deploy," then `VUE_APP_API_URL=<IP of the test VM>`
  - If the commit does not contain "test-deploy," then `VUE_APP_API_URL=/api`
- The frontend is served with NGINX.

## Test stage
Static Application Security Testing (SAST) for the entire project:
- GitLab SAST includes:
  - eslint-sast
  - gosec-sast
  - nodejs-scan-sast
  - semgrep-sast
  - spotbugs-sast (without compilation)
- SonarQube SAST


## Release stage
When the project passes tests, the container tag is changed to `latest`.
- If the commit contains "test-deploy," the frontend is uploaded to the `dumplings-frontend-test` GitLab Container repository.
- If the commit does not contain "test-deploy," the frontend is uploaded to the `dumplings-frontend` GitLab Container repository.


## Deploy to test VM
Deployment depends on the commit:
- If the commit contains "test-deploy," it deploys with Docker Compose on the test VM ([Docker Compose YAML](./docker-compose.yml)).
- If the commit contains "prod-deploy," it triggers a pipeline from the infrastructure repository for deployment to Kubernetes.
- Otherwise, the pipeline terminates - no deployment action.

## Infrastructure
The infrastructure project and its documentation can be found at this link: [Dumplings Infrastructure](https://github.com/po-khmel/dumplings-infra)

## For Developers
### Frontend

```bash
npm install
NODE_ENV=production VUE_APP_API_URL=http://localhost:8081 npm run serve
```

### Backend

```bash
go run ./cmd/api
go test -v ./... 
```
