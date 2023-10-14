# Momo Store aka Пельменная №2

<!-- <img width="900" alt="image" src="https://user-images.githubusercontent.com/9394918/167876466-2c530828-d658-4efe-9064-825626cc6db5.png"> -->

- [Momo Store aka Пельменная №2](#momo-store-aka-пельменная-2)
  - [Build](#build)
  - [Test](#test)
  - [Release](#release)
  - [Deploy to test VM](#deploy-to-test-vm)
  - [Infrastructure](#infrastructure)
  - [For Developers](#for-developers)
    - [Frontend](#frontend)
    - [Backend](#backend)

URL: https://po-khmel.space

**DEV**: Процесс сборки, тестировния, релиза и деплоя **на тестовую ВМ** сделан в GitLab CI в данном проекте.
- корневой [`.gitlab-ci.yml`](./.gitlab-ci.yml) триггерит даунстрим пайплайны: [Фронт](./frontend/.gitlab-ci.yml) и [Бэк](./backend/.gitlab-ci.yml) 


**PROD**: Процесс сборки, тестировния, релиза совпадает с DEV. Деплой в "прод" в k8s описан в проекте [DumplingInfra](https://gitlab.praktikum-services.ru/std-017-042/dumplinginfra).

## Build

[Фронт](./frontend/Dockerfile) и [Бэк](./backend/Dockerfile) билдятся в мультистейдж Dockerfile'ах
- патч версии контейнеров присваиваются по `CI_COMMIT_SHORT_SHA`
- Docker контейнеры хранятся в GitLab Container Registry
- `VUE_APP_API_URL` переменная задается через `--build-arg` и ее значение зависит от **месседжа в коммите**: 
  - коммит содержит "test-deploy" --> `VUE_APP_API_URL=<IP тестовой ВМ>`   
  - коммит **не** содержит "test-deploy" -->  `VUE_APP_API_URL=/api`  
- фронтенд раздается с NGINX

## Test
SAST для всего проекта:
- GitLab SAST:
  - eslint-sast
  - gosec-sast
  - nodejs-scan-sast
  - semgrep-sast
  - spotbugs-sast (без компиляции)
- SonarQube SAST

## Release
- проект проходит тесты --> тег контейнера меняется на `latest`
  - коммит содержит "test-deploy" --> фронтенд закидывается в репозиторий `dumplings-frontend-test`
  - коммит **не** содержит "test-deploy" --> фронтенд закидывается в репозиторий `dumplings-frontend`


## Deploy to test VM
Деплой зависит от коммита:
- коммит содержит "test-deploy" --> деплой в Docker Compose на тестовой ВМ 
- коммит содержит "prod-deploy" --> триггерится пайплайн из инфраструктурного репозитория --> деплой в k8s
- в ином случае - пайплайн завершается

## Infrastructure

Проект инфраструктуры и ее документация по ссылке : https://gitlab.praktikum-services.ru/std-017-042/dumplinginfra


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
