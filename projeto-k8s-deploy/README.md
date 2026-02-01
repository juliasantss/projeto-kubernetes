# 📚 Sistema de Mensagens Fullstack com Kubernetes (Kind)

## 👥 Equipe

| Nome | Matrícula |
|------|-----------|
| Júlia Beatriz da Silva Santos | 20231380011 |
| Luiz Philipe Lima de Andrade | 20231380035 |

---

## 🎯 Objetivo do Projeto

Este projeto implementa um sistema de mensagens simples (Frontend em React/Vite, Backend em Flask, e Database em PostgreSQL) orquestrado e exposto utilizando Kubernetes (Kind) e NGINX Ingress.

O objetivo principal é demonstrar a orquestração completa de uma aplicação web de três camadas utilizando conceitos avançados do Kubernetes, incluindo:

- **StatefulSets** para PostgreSQL com Persistent Volumes
- **Deployments** de Alta Disponibilidade para Frontend e Backend
- **ConfigMaps e Secrets** para gerenciamento de configurações e credenciais
- **Namespaces** para isolamento de ambientes (`app` e `database`)
- **Ingress** com rewrite e health checks robustos

---

## 🏗️ Arquitetura da Solução

A aplicação é dividida em três camadas distintas, gerenciadas em diferentes Namespaces:

| Camada | Tecnologia | Componente K8s | Namespace | Porta |
|--------|------------|----------------|-----------|-------|
| Frontend | React + Vite + NGINX | Deployment / Service | `app` | 80 |
| Backend (API) | Python Flask + SQLAlchemy | Deployment / Service | `app` | 5000 |
| Database | PostgreSQL | StatefulSet / PersistentVolumeClaim | `database` | 5432 |

---

## ⚙️ Passos para Deploy

### 0. Criar o Cluster Kind

Primeiro, crie o cluster Kubernetes utilizando Kind:

```bash
kind create cluster --name projetok8s
```

---

### 1. Preparação do Ambiente

Crie os namespaces e o Persistent Volume Claim (PVC) para o PostgreSQL:

```bash
# 1.1 Criar os namespaces da aplicação e do banco
kubectl apply -f namespace.yaml

# 1.2 Criar o Persistent Volume Claim para o PostgreSQL
kubectl apply -f database/postgres-pvc.yaml
```

---

### 2. Deploy do PostgreSQL (Database)

Implante o banco de dados como um StatefulSet:

```bash
# 2.1 Aplicar o Service e o StatefulSet
kubectl apply -f database/postgres-statefulset.yaml
```

**Verificação do Banco:**

Aguarde o Pod ficar `1/1 Running`:

```bash
kubectl get pods -n database

# Exemplo de saída esperada:
# NAME         READY   STATUS    RESTARTS   AGE
# postgres-0   1/1     Running   0          5m
```

---

### 3. Deploy da Aplicação (Backend e Frontend)

Aplique as configurações do Backend e Frontend, incluindo ConfigMaps e Secrets:

```bash
# 3.1 Aplicar o ConfigMap e o Secret
kubectl apply -f backend/backend-config.yaml
kubectl apply -f backend/backend-secret.yaml

# 3.2 Aplicar o Deployment do Backend e o Service
kubectl apply -f backend/deployment.yaml

# 3.3 Aplicar o Deployment do Frontend e o Service
kubectl apply -f frontend/deployment.yaml
```

**Verificação dos Pods:**

Monitore até que todos os Pods fiquem `1/1 Running`:

```bash
kubectl get pods -n app

# Exemplo de saída esperada:
# NAME                                  READY   STATUS    RESTARTS   AGE
# backend-deployment-...                1/1     Running   0          1m
# frontend-deployment-...               1/1     Running   0          1m
```

---

### 4. Instalação e Configuração do Ingress

Instale o NGINX Ingress Controller (necessário para Kind) e aplique as regras de roteamento:

```bash
# 4.1 Instalar o NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# 4.2 Aplicar as regras de Ingress
kubectl apply -f ingress/ingress.yaml
```

**Verificação do Ingress Controller:**

Obtenha o NodePort mapeado pelo Kind:

```bash
kubectl get services -n ingress-nginx

# Anote a porta NodePort (ex: 80:32671/TCP)
```

---

## 🌐 Endereços de Acesso

Como o cluster está rodando via Kind na sua Máquina Virtual (VM) Debian, o acesso deve ser feito através do IP do Container do Kind na porta NodePort mapeada.

### Encontrando o IP do Container Kind

Se o IP `172.19.0.2` não funcionar, encontre o IP correto usando:

```bash
# 1. Obter o nome do container do Kind
docker ps --filter "name=projetok8s"

# 2. Obter o IP do container
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' projetok8s-control-plane
```

### Endereços de Exemplo

**Nota:** Os valores abaixo são exemplos. Ajuste conforme seu ambiente.

- **IP do Container Kind:** `172.19.0.2` (pode variar)
- **Porta NodePort do Ingress:** `32671` (pode variar)

---

## 🔗 Testando a Aplicação

### 1. Acessar o Frontend

Abra seu navegador e acesse:

```
http://172.19.0.2:32671/
```

### 2. Testar a API (Backend)

#### Teste de Status da API

Confirma que o roteamento e a conexão com o Flask estão funcionando:

```bash
curl http://172.19.0.2:32671/api
```

**Saída Esperada:**

```json
{
  "status": "API is online",
  "endpoints": ["/api/mensagens"]
}
```

#### Teste de Health Check do Backend

```bash
curl http://172.19.0.2:32671/
```

**Saída Esperada:**

```json
{
  "status": "Backend running"
}
```

---

## 📝 Observações Importantes

### Novas Rotas no `app.py`

Foram adicionadas rotas estratégicas ao Backend para garantir a estabilidade e facilitar o diagnóstico do deploy:

#### Rota de Raiz (`/`)

- **Função:** Retorna um status de "Backend running"
- **Motivo:** Serve como um **Health Check** simplificado. Permite confirmar instantaneamente se o tráfego que entra pelo Ingress está realmente conseguindo atravessar todas as camadas até chegar ao servidor Flask.

#### Rota Base da API (`/api`)

- **Função:** Retorna o status da API e lista os endpoints ativos, como `/api/mensagens`
- **Motivo:** Foi adicionada para solucionar erros de `404 Not Found` encontrados durante os testes iniciais. Ela valida se o roteamento do Ingress configurado para o prefixo `/api` está funcionando corretamente antes de testar a persistência de dados no banco.

### Imagens Docker

As imagens da aplicação estão disponíveis no Docker Hub:

- **Backend:** `juliasantss/kube-students-backend:latest`

Certifique-se de que as imagens estejam corretamente referenciadas nos arquivos de deployment.

---

## 🛠️ Comandos Úteis

### Verificar Status dos Pods

```bash
# Pods da aplicação
kubectl get pods -n app

# Pods do banco de dados
kubectl get pods -n database
```

### Verificar Logs

```bash
# Logs do backend
kubectl logs -n app deployment/backend-deployment

# Logs do frontend
kubectl logs -n app deployment/frontend-deployment

# Logs do PostgreSQL
kubectl logs -n database postgres-0
```

### Deletar e Recriar Recursos

```bash
# Deletar todos os recursos
kubectl delete -f namespace.yaml
kubectl delete -f database/
kubectl delete -f backend/
kubectl delete -f frontend/
kubectl delete -f ingress/

# Recriar cluster
kind delete cluster --name projetok8s
kind create cluster --name projetok8s
```

---

## 📚 Recursos Adicionais

- [Documentação do Kubernetes](https://kubernetes.io/docs/)
- [Documentação do Kind](https://kind.sigs.k8s.io/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [PostgreSQL](https://www.postgresql.org/docs/)
- [Flask](https://flask.palletsprojects.com/)
- [React](https://react.dev/)

---

## 📄 Licença

Este projeto foi desenvolvido para fins acadêmicos.
