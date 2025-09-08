# Projeto GitOps na Prática com Kubernetes e ArgoCD

Este projeto demonstra a implementação de um fluxo **GitOps** para implantar uma aplicação de microsserviços (**"Online Boutique"**) em um cluster Kubernetes local.  
Utilizamos o Git como **fonte única da verdade**, e o **ArgoCD** para automatizar a sincronização entre o estado desejado no repositório e o estado real do cluster.

A aplicação é executada localmente usando o **Rancher Desktop**.  

---

## Índice
- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Etapa 1: Preparando o Repositório no GitHub](#etapa-1-preparando-o-repositório-no-github)
- [Etapa 2: Instalando o ArgoCD no Cluster](#etapa-2-instalando-o-argocd-no-cluster)
- [Etapa 3: Acessando a Interface do ArgoCD](#etapa-3-acessando-a-interface-do-argocd)
- [Etapa 4: Criando a Aplicação no ArgoCD](#etapa-4-criando-a-aplicação-no-argocd)
- [Etapa 5: Acessando o Front-end da Aplicação](#etapa-5-acessando-o-front-end-da-aplicação)
- [Opcional: Testando o Fluxo GitOps](#opcional-testando-o-fluxo-gitops)

---

## Visão Geral
O desenvolvimento moderno de aplicações exige **entregas rápidas e seguras**.  
O **GitOps** surge como uma prática para tornar os processos de deploy mais **auditáveis, previsíveis e versionados**, usando o Git como fonte da verdade para a infraestrutura e aplicações.

Neste projeto, você irá:
- Configurar um repositório no GitHub com os manifestos Kubernetes de uma aplicação de microsserviços.  
- Instalar e configurar o ArgoCD no seu cluster Kubernetes local.  
- Criar uma aplicação no ArgoCD que aponta para o seu repositório Git.  
- Sincronizar a aplicação e observar o deploy automático dos recursos no cluster.  
- Acessar a aplicação rodando localmente.  

---

## Pré-requisitos
Antes de começar, garanta que você tenha as seguintes ferramentas instaladas e configuradas:

- Rancher Desktop (com Kubernetes habilitado)  
- `kubectl` configurado (teste com `kubectl get nodes`)  
- Git instalado  
- Conta no GitHub com repositório público  
- Docker funcionando localmente  

---

## Etapa 1: Preparando o Repositório no GitHub
Vamos criar o repositório que será a **"fonte da verdade"** para o ArgoCD.

1. Faça o **Fork** do repositório original:  
   👉 [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)

2. Crie um novo repositório público no seu GitHub com o nome:  
- gitops-microservices


3. Clone o repositório para sua máquina:  
```bash
git clone https://github.com/SEU-USUARIO/gitops-microservices.git
cd gitops-microservices
```

4. Crie a estrutura de pastas e arquivo de manifesto:
```bash
mkdir k8s
cp ../microservices-demo/release/kubernetes-manifests.yaml k8s/online-boutique.yaml
```

5. Envie os arquivos para o GitHub:
```bash
git add .
git commit -m "Adiciona manifestos da aplicação Online Boutique"
git push origin main
```

## Etapa 2: Instalando o ArgoCD no Cluster

Crie o namespace e instale o ArgoCD:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verifique os pods:
```bash
kubectl get pods -n argocd
```

## Etapa 3: Acessando a Interface do ArgoCD

Expose o serviço do ArgoCD:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Abra https://localhost:8080
 e faça login:

Usuário: admin

Senha:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Etapa 4: Criando a Aplicação no ArgoCD

### Na interface do ArgoCD clique em + NEW APP e configure:

Application Name: online-boutique

Project Name: default

Sync Policy: Manual

---

### SOURCE:

Repository URL: https://github.com/SEU-USUARIO/gitops-microservices.git

Revision: HEAD

Path: k8s

---

### DESTINATION:

Cluster URL: https://kubernetes.default.svc

Namespace: default

Clique em CREATE, depois em SYNC → SYNCHRONIZE.

## Etapa 5: Acessando o Front-end da Aplicação

O frontend é exposto via ClusterIP. Use port-forward:
```bash
kubectl port-forward svc/frontend 7000:80
```
Acesse: http://localhost:7000

