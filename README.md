# Atividade Kubernetes - Nginx Olá Mundo com ArgoCD

Este projeto demonstra como criar e implantar uma aplicação web simples usando Nginx no Kubernetes com GitOps através do ArgoCD.

## Estrutura do Projeto

```
atividade_kubernetes/
├── app/
│   ├── Dockerfile      # Imagem Docker customizada do Nginx
│   └── index.html      # Página HTML "Olá Mundo"
├── k8s/
│   ├── deployment.yaml # Deployment do Kubernetes
│   └── service.yaml    # Service do Kubernetes
└── README.md          # Este arquivo
```

## Componentes

### 1. Aplicação Web
- **index.html**: Página HTML simples exibindo "Olá Mundo!"
- **Dockerfile**: Cria uma imagem Docker baseada no Nginx com a página customizada

### 2. Kubernetes Resources
- **Deployment**: Gerencia 2 réplicas da aplicação
- **Service**: Expõe a aplicação internamente no cluster

### 3. ArgoCD GitOps
- **Application**: Criada via interface do ArgoCD vinculada à branch v2

## Como Executar

### Pré-requisitos
- Docker
- Kubernetes cluster (minikube, kind, ou cluster real)
- kubectl configurado
- ArgoCD instalado no cluster

### Passos

1. **Build da imagem Docker e publicação no Docker Hub**:
   ```bash
   docker build -t seuusuario/nginx-ola-mundo:2.0 app/
   docker login
   docker push seuusuario/nginx-ola-mundo:2.0
   ```

2. **Instalar ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Configurar ArgoCD Application** (via interface):
   - Acesse a interface do ArgoCD
   - Usuário padrão: admin
   - Senha inicial pode ser descoberta em:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```
   - Crie uma nova aplicação
   - Configure o repositório Git (branch v2)
   - Defina o path como `k8s/`
   - Configure sync automático

4. **Verificar o deployment**:
   ```bash
   kubectl get pods
   kubectl get services
   argocd app list
   ```

5. **Acessar a aplicação**:
   ```bash
   kubectl port-forward service/nginx-ola-mundo-v2 8081:80
   ```
   Acesse: http://localhost:8081

## Recursos Kubernetes

### Deployment
- **Nome**: nginx-ola-mundo-v2
- **Réplicas**: 2
- **Imagem**: seuusuario/nginx-ola-mundo:2.0
- **Porta**: 80

### Service
- **Tipo**: ClusterIP
- **Porta**: 80
- **Seletor**: app=nginx-ola-mundo-v2

### ArgoCD Application
- **Criação**: Via interface do ArgoCD
- **Branch**: v2
- **Namespace**: default
- **Repositório**: Este repositório Git
- **Path**: k8s/
- **Sync Policy**: Automático

## Comandos Úteis

```bash
# Ver logs dos pods
kubectl logs -l app=nginx-ola-mundo-v2

# Escalar o deployment
kubectl scale deployment nginx-ola-mundo-v2 --replicas=3

# ArgoCD CLI
argocd app list
argocd app sync nginx-ola-mundo-v2
argocd app get nginx-ola-mundo-v2

# Acessar ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## GitOps com ArgoCD

O ArgoCD monitora a branch v2 deste repositório Git e automaticamente aplica as mudanças nos manifestos Kubernetes. Para fazer alterações:

1. Modifique os arquivos em `k8s/` na branch v2
2. Commit e push para a branch v2
3. ArgoCD detectará as mudanças e sincronizará automaticamente

## Implementação Atual

- **Branch monitorada**: v2
- **Arquivos alterados**: deployment.yaml, service.yaml, index.html
- **Configuração**: Application criada via interface do ArgoCD
- **Sincronização**: Automática com o repositório

## Benefícios do GitOps

- **Versionamento**: Histórico completo de mudanças
- **Rollback**: Fácil reversão para versões anteriores
- **Auditoria**: Rastreabilidade de todas as alterações
- **Automação**: Deploy automático baseado em Git