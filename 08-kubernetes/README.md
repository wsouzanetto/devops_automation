# Lab 01 – Criando um AKS com Terraform (Camada Gratuita)

## O Que Vamos Fazer

- Criar um cluster AKS usando Terraform
- Usar a camada gratuita do Azure (Free Tier)
- Autenticar via Azure CLI
- Aplicar a infraestrutura com comandos passo a passo
- Validar e acessar o cluster com `kubectl`

---

## Estrutura de Diretórios

```bash
aks-free-tier/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── keda.tf
├── helm-ingress.tf
```

---

1. Use the curl command to download the kubectl binary:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
This command fetches the latest stable version of kubectl and downloads the appropriate binary for the AMD64 architecture.

Give execute permission to the binary:
```bash
chmod +x kubectl
```
Move the kubectl binary to a directory in your PATH. For example, you can move it to /usr/local/bin/:
```bash
sudo mv kubectl /usr/local/bin/
```
Verify that the installation was successful by running the following command:
```bash
kubectl version --client
```

### Download the Helm installation script:

1. Download a file
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

2. Make the script executable:

```bash
chmod +x get_helm.sh
```
3. Run the installation script to install Helm:

```bash
./get_helm.sh
```
4. Verify the installation by checking the Helm version:

```bash
helm version
```

### How to install AzCli

```bash
sudo apt install azure-cli
```

### Atualizar pacotes
```bash
sudo apt update && sudo apt upgrade -y
```
###  Instalar dependências
```bash
sudo apt install -y gnupg software-properties-common curl
```

###  Adicionar chave GPG da HashiCorp
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

###  Adicionar repositório oficial
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

###  Atualizar repositórios e instalar Terraform
```bash
sudo apt update
sudo apt install -y terraform
```

# Verificar instalação
```bash
terraform -v
```

## provider.tf

```t
provider "azurerm" {
  features {}
}

terraform {
  required_version = ">= 1.0.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    kubernetes = {
      source = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
    helm = {
      source = "hashicorp/helm"
      version = "~> 2.0"
    }
  }
}

provider "kubernetes" {
  host                   = azurerm_kubernetes_cluster.aks.kube_config[0].host
  client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_certificate)
  client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_key)
  cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].cluster_ca_certificate)
}

provider "helm" {
  kubernetes {
    host                   = azurerm_kubernetes_cluster.aks.kube_config[0].host
    client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_certificate)
    client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_key)
    cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].cluster_ca_certificate)
  }
}
```

---

## variables.tf

```hcl
variable "resource_group_name" {
  default = "rg-aks-free"
}

variable "location" {
  default = "eastus"
}

variable "aks_cluster_name" {
  default = "aks-free-tier"
}
```

---

## main.tf

```t
data "azurerm_kubernetes_cluster" "aks_data" {
  name                = azurerm_kubernetes_cluster.aks.name
  resource_group_name = azurerm_kubernetes_cluster.aks.resource_group_name
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = var.aks_cluster_name
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "aksfree"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_B2s"  # ideal para camada gratuita
  }

  identity {
    type = "SystemAssigned"
  }

  tags = {
    Environment = "Dev"
  }
}

resource "azurerm_public_ip" "ingress_ip" {
  name                = "myAKSPublicIPForIngress"
  location            = azurerm_resource_group.rg.location
  resource_group_name = data.azurerm_kubernetes_cluster.aks_data.node_resource_group
  allocation_method   = "Static"
  sku                 = "Standard"
}
```

---


## helm_ingress.tf

```t
resource "kubernetes_namespace" "ingress" {
  metadata {
    name = "ingress-nginx"
  }
}

resource "helm_release" "nginx_ingress" {
  name       = "ingress-nginx"
  namespace  = kubernetes_namespace.ingress.metadata[0].name
  repository = "https://kubernetes.github.io/ingress-nginx"
  chart      = "ingress-nginx"
  version    = "4.9.1"

  values = [
    <<EOF
controller:
  replicaCount: 2
  nodeSelector:
    beta.kubernetes.io/os: linux
  service:
    type: LoadBalancer
    externalTrafficPolicy: Local
    loadBalancerIP: "${azurerm_public_ip.ingress_ip.ip_address}"

defaultBackend:
  nodeSelector:
    beta.kubernetes.io/os: linux
EOF
  ]

  depends_on = [azurerm_public_ip.ingress_ip]
}

```

## helm_keda.tf

```t
resource "kubernetes_namespace" "keda" {
  metadata {
    name = "keda"
  }
}

resource "helm_release" "keda" {
  name       = "keda"
  namespace  = kubernetes_namespace.keda.metadata[0].name
  repository = "https://kedacore.github.io/charts"
  chart      = "keda"
  version    = "2.13.2"

  values = [
    <<EOF
replicaCount: 1
EOF
  ]

  depends_on = [kubernetes_namespace.keda]
}

```

## outputs.tf

```t

output "next_steps" {
  value = <<EOT
🎉 Cluster AKS criado com sucesso!

👉 Para acessar seu cluster, execute o seguinte comando:

az aks get-credentials --resource-group ${var.resource_group_name} --name ${var.aks_cluster_name}

🚀 Ingress Controller instalado com sucesso!

✅ Acesse o IP fixo criado:
http://${azurerm_public_ip.ingress_ip.ip_address}

📄 Você verá uma mensagem "404 Not Found" do NGINX — isso é esperado!

➡️ Prossiga para o Lab 03 para criar uma aplicação + Ingress Resource.

EOT
}

output "resource_group_name" {
  value = var.resource_group_name
}

output "aks_cluster_name" {
  value = var.aks_cluster_name
}
```

---

## Passo a Passo para Rodar

### Pré-requisitos

- Instalar [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- Instalar [Terraform](https://developer.hashicorp.com/terraform/downloads)
- Ter uma conta Azure com créditos ou Free Tier

---

### 1. Login no Azure

```bash
az login
```

---

### 2. Inicializar o Terraform

```bash
terraform init
```

---

### 3. Visualizar o plano de execução

```bash
terraform plan
```

---

### 4. Aplicar a infraestrutura

```bash
terraform apply -auto-approve
```

---

### 5. Acessar o cluster AKS

```bash
az aks get-credentials --resource-group rg-aks-free --name aks-free-tier
kubectl get nodes
```

# Lab 2 - Criando seu primeiro Pod no Kubernetes (porta 8081)

## Objetivo

Criar manualmente um Pod com a imagem `iesodias/java-api:latest`, rodando na porta `8081`, e interagir com ele via `kubectl`.

---

## Passo a Passo

### 1. Criar a estrutura do diretório

```bash
mkdir -p ~/k8s-workshop/lab1
cd ~/k8s-workshop/lab1
```

---

### 2. Criar o manifesto do Pod

```bash
nano pod-java-api.yaml
```

Cole o conteúdo abaixo no arquivo, com a porta correta:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: java-api-pod
spec:
  containers:
    - name: java-api
      image: iesodias/java-api:latest
      ports:
        - containerPort: 8081
```

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f pod-java-api.yaml
```

---

### 4. Verificar o estado do Pod

```bash
kubectl get pods
```

---

### 5. Interagir com o Pod

```bash
# Ver os logs do container
kubectl logs java-api-pod

# Acessar shell (se a imagem permitir)
kubectl exec -it java-api-pod -- sh

# Ver detalhes do Pod
kubectl describe pod java-api-pod
```

---

### 6. Testar a aplicação localmente

Use o `port-forward` para acessar a porta 8081 localmente:

```bash
kubectl port-forward pod/java-api-pod 8081:8081
```

Abra outro terminal e teste com `curl`:

```bash
curl http://localhost:8081
```

---

### 7. Limpeza do Ambiente (opcional, mas recomendado)

Após terminar os testes, você pode deletar o Pod para manter seu cluster limpo:

```bash
kubectl delete -f pod-java-api.yaml
```

Ou, alternativamente, pelo nome do Pod:

```bash
kubectl delete pod java-api-pod
```

> Reforce no workshop que no mundo real, geralmente não se cria Pods “na mão” — isso é apenas didático.

---

## Resultado Esperado

* Pod rodando a imagem `iesodias/java-api:latest`
* Porta `8081` redirecionada localmente
* Aplicação respondendo via `curl`

---


# Lab 3 – Usando ReplicaSet para manter múltiplos Pods

## Objetivo

* Criar um ReplicaSet com a imagem `iesodias/java-api:latest`
* Observar a criação automática de réplicas
* Aumentar e diminuir a quantidade de réplicas
* Deletar Pods manualmente e observar o ReplicaSet restaurá-los
* Finalizar com a exclusão dos recursos

---

## Passo a Passo

### 1. Criar o diretório do lab

```bash
mkdir -p ~/k8s-workshop/lab2
cd ~/k8s-workshop/lab2
```

---

### 2. Criar o manifesto do ReplicaSet

```bash
vi replicaset-java-api.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: java-api-replicaset
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
```

> Note o `selector.matchLabels` e os `labels` no `template`. Isso é crucial para o ReplicaSet controlar os Pods.

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f replicaset-java-api.yaml
```

---

### 4. Verificar as réplicas criadas

```bash
kubectl get pods -l app=java-api
kubectl get rs
```

---

### 5. Testar o comportamento do ReplicaSet

#### Aumentar a quantidade de réplicas:

```bash
kubectl scale rs java-api-replicaset --replicas=4
kubectl get pods -l app=java-api
```

#### Diminuir a quantidade de réplicas:

```bash
kubectl scale rs java-api-replicaset --replicas=1
kubectl get pods -l app=java-api
```

#### Apagar manualmente um Pod e observar a auto-recuperação:

```bash
kubectl get pods
kubectl delete pod <nome-de-um-pod>
kubectl get pods -l app=java-api -w
```

> O ReplicaSet vai detectar a ausência e recriar o Pod automaticamente.

---

### 6. Testar um dos Pods localmente

Use `kubectl get pods` para pegar o nome de um Pod válido e:

```bash
kubectl port-forward pod/<nome-do-pod> 8081:8081
curl http://localhost:8081
```

---

### 7. Limpeza do Ambiente

```bash
kubectl delete -f replicaset-java-api.yaml
```

---

## Resultado Esperado

* Entender o que é um ReplicaSet
* Visualizar a criação automática de réplicas
* Escalar horizontalmente (up/down)
* Ver o comportamento de autocorreção ao deletar Pods
* Encerrar limpando todos os recursos


# Lab 4 – Deployment com réplicas, rolling updates e rollback

## Objetivo

* Criar um Deployment com múltiplas réplicas
* Fazer rolling update alterando a imagem
* Observar como o Deployment gerencia ReplicaSets
* Testar escalabilidade horizontal
* Simular um erro e realizar rollback
* Acessar a aplicação localmente
* Deletar todos os recursos ao final

---

## Passo a Passo

### 1. Criar o diretório do lab

```bash
mkdir -p ~/k8s-workshop/lab3
cd ~/k8s-workshop/lab3
```

---

### 2. Criar o manifesto do Deployment

```bash
vi deployment-java-api.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
```

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f deployment-java-api.yaml
```

---

### 4. Verificar o Deployment, ReplicaSets e Pods

```bash
kubectl get deployments
kubectl get rs
kubectl get pods -l app=java-api
```

---

### 5. Testar um dos Pods via port-forward

```bash
kubectl get pods -l app=java-api
kubectl port-forward pod/<nome-do-pod> 8081:8081
curl http://localhost:8081
```

---

### 6. Escalar o Deployment

```bash
kubectl scale deployment java-api-deployment --replicas=4
kubectl get pods -l app=java-api
```

---

### 7. Atualizar a imagem com uma tag errada (simulando falha)

```bash
kubectl set image deployment java-api-deployment java-api=iesodias/java-api:errada
```

Acompanhe o rollout:

```bash
kubectl rollout status deployment java-api-deployment
kubectl get pods
```

Você verá falhas no rollout (Pods em `CrashLoopBackOff` ou `ImagePullBackOff`).

---

### 8. Fazer rollback da atualização com falha

```bash
kubectl rollout undo deployment java-api-deployment
kubectl rollout status deployment java-api-deployment
```

---

### 9. Ver histórico de rollouts

```bash
kubectl rollout history deployment java-api-deployment
```

---

### 10. Limpeza total dos recursos criados

```bash
kubectl delete -f deployment-java-api.yaml
```

---

## Resultado Esperado

* Deployment com 2+ réplicas criado e funcional
* Escalado para 4 Pods
* Rolling update com erro simulado
* Rollback executado com sucesso
* Acesso local via port-forward
* Recursos removidos ao final
 
---

# Lab 5 – Expondo sua aplicação via LoadBalancer no AKS (Manifesto Único)

## Objetivo

* Criar um Deployment com múltiplos Pods da aplicação Java
* Expor com um Service do tipo LoadBalancer
* Acessar a aplicação via navegador e curl
* Ver o balanceamento entre os Pods
* Encerrar com a limpeza dos recursos

---

## Passo a Passo

### 1. Criar a estrutura de diretórios

```bash
mkdir -p ~/k8s-workshop/lab4
cd ~/k8s-workshop/lab4
```

---

### 2. Criar o manifesto combinado

```bash
vi java-api-lb.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-service
spec:
  type: LoadBalancer
  selector:
    app: java-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8081
```

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f java-api-lb.yaml
```

---

### 4. Aguardar o IP público ser atribuído

```bash
kubectl get service java-api-service
```

Aguarde até ver um valor válido no campo `EXTERNAL-IP`, por exemplo:

```bash
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
java-api-service    LoadBalancer   10.0.128.34    52.170.42.120   80:80/TCP      2m
```

---

### 5. Acessar a aplicação no navegador

Abra o navegador e acesse:

```
http://<EXTERNAL-IP>
```

Exemplo:

```
http://52.170.42.120
```

Você deve ver a resposta da API Java.

---

### 6. Testar com curl

```bash
curl http://<EXTERNAL-IP>
```

---

### 7. Testar o balanceamento entre Pods

Se sua aplicação retornar alguma identificação (como hostname), execute:

```bash
watch -n 1 curl http://<EXTERNAL-IP>
```

---

### 8. Escalar o Deployment

```bash
kubectl scale deployment java-api-deployment --replicas=5
kubectl get pods
```

---

### 9. Deletar um Pod e observar a recuperação

```bash
kubectl delete pod <nome-de-um-pod>
kubectl get pods -w
```

---

### 10. Limpeza do Lab

```bash
kubectl delete -f java-api-lb.yaml
```

---

## Resultado Esperado

* Aplicação acessível via IP público e navegador
* Service do tipo LoadBalancer gerando IP externo no AKS
* Escalabilidade e alta disponibilidade validadas
* Ambiente limpo após finalização

---

# Lab 6 – Namespaces no Kubernetes: organizando ambientes dev, hml e prod

## Objetivo

* Compreender o papel dos Namespaces no Kubernetes
* Criar ambientes isolados: dev, hml e prod
* Implantar a mesma aplicação nos três ambientes
* Acessar individualmente os Pods em cada namespace
* Observar o isolamento de recursos
* Limpar os namespaces e seus objetos

---

## Passo a Passo

### 1. Criar o diretório do lab

```bash
mkdir -p ~/k8s-workshop/lab5
cd ~/k8s-workshop/lab5
```

---

### 2. Criar um único manifesto com os três ambientes

```bash
vi namespaces-multi-env.yaml
```

Conteúdo completo:

```yaml
# Criação dos namespaces
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: hml
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
---
# Deployment no namespace dev
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        env: dev
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
# Deployment no namespace hml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: hml
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        env: hml
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
# Deployment no namespace prod
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
        env: prod
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
```

---

### 3. Aplicar todos os recursos

```bash
kubectl apply -f namespaces-multi-env.yaml
```

---

### 4. Ver os namespaces criados

```bash
kubectl get namespaces
```

---

### 5. Listar os Pods por namespace

```bash
kubectl get pods -n dev
kubectl get pods -n hml
kubectl get pods -n prod
```

---

### 6. Acessar um Pod de cada ambiente

```bash
kubectl exec -n dev -it $(kubectl get pods -n dev -o jsonpath='{.items[0].metadata.name}') -- sh
```

Repita o comando para os namespaces `hml` e `prod`.

---

### 7. Comparar logs nos ambientes

```bash
kubectl logs -n dev -l app=java-api
kubectl logs -n hml -l app=java-api
kubectl logs -n prod -l app=java-api
```

---

### 8. Simular uma falha apenas em dev

```bash
kubectl delete pod -n dev -l app=java-api
kubectl get pods -n dev -w
```

> O ReplicaSet no ambiente dev irá recriar o Pod automaticamente. Os demais ambientes não são afetados.

---

### 9. Criar um Service por namespace (opcional)

```bash
kubectl expose deployment java-api --port=80 --target-port=8081 -n dev
```

Acesse via port-forward:

```bash
kubectl port-forward -n dev service/java-api 8081:80
```

---

### 10. Limpeza completa do lab

```bash
kubectl delete namespace dev
kubectl delete namespace hml
kubectl delete namespace prod
```

> Isso remove automaticamente todos os objetos de cada namespace.

---

## Resultado Esperado

* Três namespaces (dev, hml, prod) ativos
* Mesma aplicação rodando isoladamente em cada ambiente
* Acesso separado e logs por namespace
* Recriação automática de Pods apenas no ambiente afetado
* Limpeza completa ao final

---

# Lab 7 – Request & Limit de CPU e Memória

## Objetivo

* Criar um Deployment com limites de CPU e memória
* Observar como o Kubernetes reserva e limita recursos
* Ajustar os valores e ver o efeito no pod
* Verificar as métricas com `kubectl top`
* Finalizar com a exclusão dos recursos

---

## Passo a Passo

### 1. Criar o diretório do lab

```bash
mkdir -p ~/k8s-workshop/lab6
cd ~/k8s-workshop/lab6
```

---

### 2. Criar o manifesto com requests e limits

```bash
vi java-api-limited.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-limited
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api-limited
  template:
    metadata:
      labels:
        app: java-api-limited
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
```

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f java-api-limited.yaml
```

---

### 4. Ver o pod rodando

```bash
kubectl get pods
kubectl describe pod -l app=java-api-limited
```

---

### 5. Verificar o uso com kubectl top

> Se estiver no AKS, o Metrics Server já vem habilitado

```bash
kubectl top pod -l app=java-api-limited
```

Exemplo de saída:

```
NAME                                  CPU(cores)   MEMORY(bytes)
java-api-limited-xxxxxxxxxx-xxxxx     12m          90Mi
```

---

### 6. Simular carga no pod (opcional)

> Se a imagem suportar gerar carga ou houver um sidecar configurado, o uso aumentará até atingir os limites.
> Caso contrário, discuta o conceito com os alunos.

---

### 7. Aumentar os limites via kubectl edit

```bash
kubectl edit deployment java-api-limited
```

Altere os valores para:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

Após salvar:

```bash
kubectl rollout status deployment java-api-limited
```

---

### 8. Ver novo uso com kubectl top

```bash
kubectl top pod -l app=java-api-limited
```

---

### 9. Verificar a QoS atribuída ao Pod

```bash
kubectl get pod -l app=java-api-limited -o jsonpath="{.items[0].status.qosClass}"
```

Você verá:

```
Burstable
```

### 10. Limpeza do Lab

```bash
kubectl delete -f java-api-limited.yaml
```

---

## Resultado Esperado

* Pod com requests e limits aplicados
* Uso de CPU/memória monitorado com `kubectl top`
* Ajustes de limites aplicados com sucesso
* Compreensão da QoS (Quality of Service) do Kubernetes
* Recursos limpos após finalização

---

# Lab 8 – Health Checks com liveness e readiness probes (com simulação de falha)

## Objetivo

* Adicionar probes à aplicação Java (liveness e readiness)
* Ver comportamento de restart automático via liveness
* Ver remoção de Pods do tráfego via readiness
* Simular falha manual via kubectl edit
* Restauração e limpeza

---

## Passo a Passo

### 1. Criar a estrutura do Lab

```bash
mkdir -p ~/k8s-workshop/lab7
cd ~/k8s-workshop/lab7
```

---

### 2. Criar o manifesto com probes

```bash
vi java-api-health.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-health
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api-health
  template:
    metadata:
      labels:
        app: java-api-health
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8081
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8081
            initialDelaySeconds: 5
            periodSeconds: 3
            failureThreshold: 2
```

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f java-api-health.yaml
```

---

### 4. Verificar o status do Pod e das probes

```bash
kubectl get pods
kubectl describe pod -l app=java-api-health
```

---

### 5. Testar o endpoint manualmente com curl

```bash
kubectl port-forward pod/$(kubectl get pod -l app=java-api-health -o jsonpath='{.items[0].metadata.name}') 8081:8081
```

Em outro terminal:

```bash
curl http://localhost:8081/actuator/health
```

Resposta esperada:

```json
{"status":"UP"}
```

---

### 6. Simular falha de liveness editando o deployment

```bash
kubectl edit deployment java-api-health
```

Altere:

```yaml
livenessProbe:
  httpGet:
    path: /actuator/404
```

Salve e saia do editor.

---

### 7. Observar o comportamento do Kubernetes

```bash
kubectl get pods -l app=java-api-health -w
kubectl describe pod -l app=java-api-health
```

Mensagens esperadas:

```
Liveness probe failed: HTTP probe failed with statuscode: 404
Back-off restarting failed container
```

---

### 8. Restaurar o endpoint correto

```bash
kubectl edit deployment java-api-health
```

Volte o `path` da liveness para `/actuator/health`.

---

### 9. Verificar se o pod voltou ao normal

```bash
kubectl get pods -l app=java-api-health
```

O Pod deve estar com `READY 1/1` e `STATUS Running`.

---

### 10. Limpeza do Lab

```bash
kubectl delete -f java-api-health.yaml
```

---

## Resultado Esperado

* Health checks ativos com sucesso
* Falha em liveness gerou reinício automático do container
* Readiness removeu o Pod temporariamente do tráfego
* Aplicação restaurada após correção do path
* Ambiente limpo ao final

---

## Dicas para Discussão

| Conceito     | Liveness               | Readiness                 |
| ------------ | ---------------------- | ------------------------- |
| Quando falha | Container é reiniciado | Pod é removido do Service |
| Finalidade   | Detectar travamento    | Aguardar app estar pronta |
| Exemplo      | deadlock, thread stuck | carregamento de cache     |


# Lab 9 – HPA com Sidecar: Autoescalando com base em carga gerada via stress

## Objetivo

* Criar um Deployment com dois containers:

  * `java-api`: aplicação principal
  * `stress`: sidecar que gera uso de CPU
* Aplicar um HPA que escala com base na CPU
* Ver o comportamento de escalonamento em tempo real
* Fazer limpeza ao final

---

## Passo a Passo

### 1. Criar a estrutura do Lab

```bash
mkdir -p ~/k8s-workshop/lab8
cd ~/k8s-workshop/lab8
```

---

### 2. Criar o manifesto com Deployment + HPA + Sidecar

```bash
vi java-api-hpa.yaml
```

Conteúdo completo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-hpa
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api-hpa
  template:
    metadata:
      labels:
        app: java-api-hpa
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "500m"
        - name: stress
          image: progrium/stress
          command: ["stress"]
          args: ["--cpu", "1"]
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "300m"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-api-hpa
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

### 3. Aplicar o manifesto

```bash
kubectl apply -f java-api-hpa.yaml
```

---

### 4. Verificar se o Deployment e HPA estão funcionando

```bash
kubectl get deployment
kubectl get pods
kubectl get hpa
```

Exemplo de saída:

```
NAME             TARGETS   MINPODS   MAXPODS   REPLICAS
java-api-hpa     72%/50%    1         5         2
```

---

### 5. Observar o escalonamento em tempo real

```bash
watch kubectl get hpa
```

E em paralelo:

```bash
watch kubectl get pods -l app=java-api-hpa
```

> Conforme o stress consome CPU, o número de réplicas aumenta automaticamente.

---

### 6. Simular a redução de carga

Edite o Deployment:

```bash
kubectl edit deployment java-api-hpa
```

Apague a seção do container `stress` e salve.

> Isso fará com que os novos Pods venham sem carga. O HPA verá a queda no uso de CPU e irá reduzir as réplicas.

---

### 7. Aguarde o HPA escalar para baixo

```bash
watch kubectl get hpa
```

Exemplo de saída:

```
NAME             TARGETS   MINPODS   MAXPODS   REPLICAS
java-api-hpa     15%/50%    1         5         1
```

---

### 8. Limpeza do Lab

```bash
kubectl delete -f java-api-hpa.yaml
```

---

## Resultado Esperado

* O Deployment escala automaticamente de 1 até 5 Pods conforme a carga
* O sidecar `stress` gera CPU suficiente para ativar o HPA
* Remoção do sidecar reduz a carga e os Pods diminuem automaticamente
* Limpeza completa ao final

# Lab 10 – Expondo a aplicação via Ingress com nip.io

## Objetivo

* Expor sua aplicação Java no AKS usando Ingress
* Utilizar domínio dinâmico com nip.io vinculado ao IP fixo do Ingress
* Testar acesso direto via navegador

---

## Passo a Passo

### 1. Criar a estrutura do lab

```bash
mkdir -p ~/k8s-workshop/lab9
cd ~/k8s-workshop/lab9
```

---

### 2. Descobrir o IP fixo do LoadBalancer do Ingress

```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

Saída esperada:

```
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)
ingress-nginx-controller   LoadBalancer   10.0.100.10   20.210.50.123    80:80/TCP
```

Anote o EXTERNAL-IP. Vamos usá-lo no próximo passo.

---

### 3. Criar o manifesto com Deployment + Service + Ingress (nip.io)

```bash
vi java-api-ingress-nip.yaml
```

Substitua `20.210.50.123` pelo IP real do seu Ingress Controller.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-service
  namespace: default
spec:
  selector:
    app: java-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8081
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: java-api-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: java-api.20.210.50.123.nip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: java-api-service
                port:
                  number: 80
```

---

### 4. Aplicar o manifesto

```bash
kubectl apply -f java-api-ingress-nip.yaml
```

---

### 5. Acessar no navegador

Abra no navegador:

```
http://java-api.<SEU_IP>.nip.io
```

✔️ A aplicação Java deve carregar corretamente.

---

### 6. Testar com curl (opcional)

```bash
curl http://java-api.<SEU_IP>.nip.io
```

---

### 7. Limpeza do Lab

```bash
kubectl delete -f java-api-ingress-nip.yaml
```

---

## Resultado Esperado

* Aplicação Java disponível via Ingress + domínio nip.io
* Acesso direto no navegador sem necessidade de alterar o /etc/hosts
* Manifesto 100% reutilizável e dinâmico

# Lab 11 – Autoescalonamento com KEDA via Cron (Scale-to-Zero)

## Objetivo

* Configurar escalonamento automático usando agendamento via cron
* Testar scale-to-zero real (sem pods fora da janela)
* Validar escalonamento previsível com base em horário (ótimo para workshops e produção)

---

## Como calcular UTC ⇄ seu horário local

Antes de configurar os horários no manifesto, verifique a hora atual em UTC e converta para o seu horário local.

### Fórmula:

```bash
Horário local = UTC - 3
UTC = Horário local + 3
```

> Use `date -u` para ver o horário atual em UTC e ajustar os campos `start` e `end` do cron

---

## Estrutura do Lab

```bash
mkdir -p ~/k8s-workshop/lab11
cd ~/k8s-workshop/lab11
```

---

## Criar o manifesto java-api-keda-cron.yaml

```bash
vi java-api-keda-cron.yaml
```

Conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-keda
  labels:
    app: java-api-keda
spec:
  replicas: 0
  selector:
    matchLabels:
      app: java-api-keda
  template:
    metadata:
      labels:
        app: java-api-keda
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8081
            initialDelaySeconds: 20
            periodSeconds: 5
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-keda
spec:
  selector:
    app: java-api-keda
  ports:
    - port: 80
      targetPort: 8081
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: java-api-scaler
spec:
  scaleTargetRef:
    name: java-api-keda
  minReplicaCount: 0
  maxReplicaCount: 5
  cooldownPeriod: 30
  triggers:
    - type: cron
      metadata:
        timezone: UTC
        start: "56 3 * * *"
        end: "0 4 * * *"
        desiredReplicas: "2"
```

---

## Aplicar o manifesto

```bash
kubectl apply -f java-api-keda-cron.yaml
```

---

## Testando o comportamento

### 1. Antes do horário

```bash
kubectl get pods
```

> Deve mostrar nenhum pod (replicas = 0)

### 2. Durante a janela (ex: 03:56 UTC)

```bash
watch kubectl get pods
```

> Você verá 2 pods subindo automaticamente

### 3. Após a janela (ex: 04:00 UTC)

> Os pods serão removidos automaticamente, retornando ao estado "scale-to-zero"

---

## Verificando os recursos do KEDA

```bash
kubectl get scaledobject
kubectl describe scaledobject java-api-scaler
kubectl get hpa
```

> Você verá:
>
> * HPA gerado automaticamente pelo KEDA
> * Status Ready = True
> * Comportamento programado funcionando

---

## Limpeza do Lab

```bash
kubectl delete -f java-api-keda-cron.yaml
```

---

## Resultado Esperado

| Estado           | Resultado                      |
| ---------------- | ------------------------------ |
| Antes do horário | 0 pods                         |
| Durante janela   | 2 pods em execução             |
| Após a janela    | Retorno automático para 0 pods |

---

## Dicas para Discussão em Workshop

| Conceito               | Valor didático                   |
| ---------------------- | -------------------------------- |
| Escalonamento agendado | Perfeito para tarefas periódicas |
| Scale-to-zero real     | Economia de recursos             |
| Cron previsível        | Ideal para demonstrar controle   |
| `date -u`              | Mostrar que o cluster usa UTC    |
