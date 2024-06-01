# Criando-um-Pipeline-de-Deploy-de-uma-Aplicação-Utilizando-Gitlab-Docker-e-Kubernetes

## Introdução
Neste artigo, exploraremos como criar um pipeline de deploy automatizado para uma aplicação Node.js usando Gitlab CI/CD, Docker e Kubernetes. O objetivo é configurar dois cenários de deploy: um para execução da aplicação em um container Docker e outro para execução em um cluster Kubernetes hospedado no Google Cloud Platform (GCP).

## Módulo 1: Preparação do Ambiente

### Gitlab e Repositório
Inicialmente, é necessário configurar um repositório no Gitlab e preparar duas branches:
- `feature/docker-deploy`: para o deploy em Docker.
- `feature/kubernetes-deploy`: para o deploy em Kubernetes.

### Docker
Para o deploy em Docker, criaremos um `Dockerfile` na branch `feature/docker-deploy`:

```Dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### Kubernetes
Para o deploy em Kubernetes, precisaremos de um arquivo de configuração `deployment.yaml` na branch `feature/kubernetes-deploy`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
    spec:
      containers:
      - name: nodejs
        image: gcr.io/[PROJECT_ID]/nodejs-app:latest
        ports:
        - containerPort: 3000
```

## Módulo 2: Configuração do Pipeline no Gitlab CI/CD

### Pipeline para Docker
No arquivo `.gitlab-ci.yml` da branch `feature/docker-deploy`, definiremos as etapas do pipeline:

```yaml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - docker build -t my-nodejs-app .
    - docker push my-nodejs-app

deploy:
  stage: deploy
  script:
    - docker run -d -p 3000:3000 my-nodejs-app
```

### Pipeline para Kubernetes
No arquivo `.gitlab-ci.yml` da branch `feature/kubernetes-deploy`, configuraremos o pipeline para Kubernetes:

```yaml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - docker build -t gcr.io/[PROJECT_ID]/nodejs-app:latest .
    - gcloud auth configure-docker
    - docker push gcr.io/[PROJECT_ID]/nodejs-app:latest

deploy:
  stage: deploy
  script:
    - gcloud container clusters get-credentials [CLUSTER_NAME] --zone [ZONE] --project [PROJECT_ID]
    - kubectl apply -f deployment.yaml
```

## Conclusão
Com esses passos, você terá um pipeline de CI/CD robusto que automatiza o deploy de uma aplicação Node.js tanto em um container Docker quanto em um cluster Kubernetes. Lembre-se de substituir `[PROJECT_ID]`, `[CLUSTER_NAME]` e `[ZONE]` pelos valores correspondentes ao seu projeto no GCP.

Espero que este artigo tenha sido útil para entender como estruturar e configurar um pipeline de deploy utilizando as ferramentas Gitlab, Docker e Kubernetes.
