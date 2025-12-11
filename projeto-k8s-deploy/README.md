📚 README: Sistema de Mensagens Fullstack com Kubernetes (Kind)
Este projeto implementa um sistema de mensagens simples (Frontend em React/Vite, Backend em Flask, e Database em PostgreSQL) orquestrado e exposto utilizando Kubernetes (Kind) e NGINX Ingress.

Nome: Júlia Beatriz da Silva Santos
Matrícula: 20231380011

Nome: Luiz Philipe Lima de Andrade
Matrícula: 20231380035

🎯 Objetivo do Projeto
O objetivo principal é demonstrar a orquestração completa de uma aplicação web de três camadas (Frontend, Backend, Database) utilizando conceitos avançados do Kubernetes, incluindo:

- StatefulSets (PostgreSQL) e Persistent Volumes.
- Deployments de Alta Disponibilidade (Frontend e Backend).
- ConfigMaps e Secrets para gerenciamento de configurações e credenciais.
- Namespaces para isolamento de ambientes (app e database).
- Ingress com rewrite e health checks robustos.

🏗️ Arquitetura da Solução
A aplicação é dividida em três camadas distintas, gerenciadas em diferentes Namespaces, conforme o diagrama:

Camada,Tecnologia,Componente K8s,Namespace,Porta
Frontend,React + Vite + NGINX,Deployment / Service,app,80
Backend (API),Python Flask + SQLAlchemy,Deployment / Service,app,5000
Database,PostgreSQL,StatefulSet / PersistentVolumeClaim,database,5432



⚙️ Passos para Aplicação (Deploy)
Subir as imagens no Docker Hub (juliasantss/kube-students-backend:latest)
siga os passos abaixo no terminal da sua Máquina Virtual (Debian) para aplicar o projeto.
-----------------------------------------------------------------------------------------------------

1. Preparação do Ambiente
Crie os namespaces e o Persistent Volume Claim (PVC) para o PostgreSQL.
# 1.1 Criar o namespace da aplicação e do banco
kubectl apply -f namespace.yaml

# 1.2 Criar o Persistent Volume Claim para o PostgreSQL
kubectl apply -f database/postgres-pvc.yaml
-----------------------------------------------------------------------------------------------------

2. Deploy do PostgreSQL (Database)
Implante o banco de dados como um StatefulSet.
# 2.1 Aplicar o Service e o StatefulSet
kubectl apply -f database/postgres-statefulset.yaml

Verificação do Banco: Aguarde o Pod ficar 1/1 Running.
kubectl get pods -n database
# Exemplo de saída: postgres-0   1/1     Running   0          5m
-----------------------------------------------------------------------------------------------------
3. Deploy da Aplicação (Backend e Frontend)
Aplique as configurações do Backend e Frontend, incluindo ConfigMaps e Secrets.

# 3.1 Aplicar o ConfigMap (com o DB_HOST corrigido) e o Secret
kubectl apply -f backend/backend-config.yaml
kubectl apply -f backend/backend-secret.yaml

# 3.2 Aplicar o Deployment do Backend e o Service
kubectl apply -f backend/deployment.yaml

# 3.3 Aplicar o Deployment do Frontend e o Service
kubectl apply -f frontend/deployment.yaml

Verificação dos Pods: Monitore até que todos os Pods fiquem 1/1 Running.
kubectl get pods -n app
# Exemplo de saída:
# backend-deployment-...   1/1     Running   0          1m
# frontend-deployment-...  1/1     Running   0          1m
-----------------------------------------------------------------------------------------------------
4. Instalação e Configuração do Ingress
Instale o NGINX Ingress Controller (necessário para Kind) e aplique as regras de roteamento.

# 4.1 Instalar o NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# 4.2 Aplicar as regras de Ingress
kubectl apply -f ingress/ingress.yaml

Verificação do Ingress Controller: Obtenha o NodePort mapeado pelo Kind.
kubectl get services -n ingress-nginx
# Anote a porta 3XXXX (ex: 80:32671/TCP)
-----------------------------------------------------------------------------------------------------

🌐 Endereços de Acesso Esperados
Como o cluster está rodando via Kind na sua Máquina Virtual (VM) Debian, o acesso deve ser feito através do IP do Container do Kind na porta NodePort mapeada.

Endereços Encontrados no Histórico
IP do Container Kind: 172.19.0.2 (Pode variar)
Porta NodePort do Ingress: 32671 (Pode variar)
-----------------------------------------------------------------------------------------------------
1. Acesso à Aplicação (Frontend)
Acesso via navegador para o Frontend:
http://172.19.0.2:32671/
-----------------------------------------------------------------------------------------------------
3. Acesso e Teste da API (Backend)
Teste a rota de status da API (confirma que o roteamento e a conexão com o Flask estão OK):
curl http://172.19.0.2:32671/api
# Saída Esperada (Confirma a funcionalidade da API):
# {"endpoints":["/api/mensagens"],"status":"API is online"}
-----------------------------------------------------------------------------------------------------
Observação: Se o IP 172.19.0.2 não funcionar, encontre o IP correto usando:
# 1. Obter o nome do container do Kind
docker ps --filter "name=projetok8s"
# 2. Obter o IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' projetok8s-control-plane








