# Lab: Monitorando AKS com Datadog (Free Tier)

Este lab mostra como criar um cluster AKS no Free Tier e configurar o agente do Datadog corretamente para coletar métricas, logs e visualizar tudo no painel da Datadog.

---

## ✅ Etapa 1 — Criar Resource Group no Azure
```bash
az group create \
  --name aks-free-rg \
  --location eastus
```

---

## ☸️ Etapa 2 — Criar Cluster AKS (Free Tier)
```bash
az aks create \
  --resource-group aks-free-rg \
  --name aks-free-cluster \
  --node-count 1 \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --enable-managed-identity \
  --tier free \
  --location eastus
```

---

## 🔑 Etapa 3 — Acessar o Cluster via kubectl
```bash
az aks get-credentials \
  --resource-group aks-free-rg \
  --name aks-free-cluster \
  --overwrite-existing
```

---

## 📦 Etapa 4 — Adicionar repositório Helm do Datadog
```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

---

## 🔐 Etapa 5 — Criar Secret com API Key da Datadog
```bash
kubectl create namespace datadog

kubectl create secret generic datadog-secret \
  --from-literal=api-key=<SUA_API_KEY_DD> \
  -n datadog
```

---

## ⚙️ Etapa 6 — Criar arquivo `datadog-values.yaml`

Crie um arquivo chamado `datadog-values.yaml` com o seguinte conteúdo:

```yaml
datadog:
  apiKeyExistingSecret: "datadog-secret"
  clusterName: "aks-free-cluster"
  logs:
    enabled: true
    containerCollectAll: true
  hostnameForceConfigAsCanonical: false
  kubelet:
    tlsVerify: false

agents:
  useHostPID: true

rbac:
  create: true
  serviceAccount:
    create: true

clusterAgent:
  enabled: true
  replicas: 1
  service:
    enabled: true
  metricsProvider:
    enabled: true
  createPodDisruptionBudget: true
```

---

## 🚀 Etapa 7 — Instalar o Datadog com Helm
```bash
helm install datadog-agent datadog/datadog \
  -f datadog-values.yaml \
  -n datadog
```

> ⚠️ Se já tiver instalado antes:
```bash
helm upgrade datadog-agent datadog/datadog \
  -f datadog-values.yaml \
  -n datadog
```

---

## ✅ Verificações após o deploy

### Ver pods ativos:
```bash
kubectl get pods -n datadog
```

### Ver logs do agente:
```bash
kubectl logs -n datadog -l app=datadog-agent
```

### Ver logs do Cluster Agent:
```bash
kubectl logs -n datadog -l app=datadog-cluster-agent
```

---

## 🧼 Limpar host antigo (opcional)
Se tiver hostname antigo (ex: `aks-nodepool...`), vá no painel do Datadog → Infrastructure e clique em "Delete Host" no nó antigo (em cinza).

---

Pronto! O cluster AKS está 100% integrado ao Datadog com métricas, logs e painel interativo. Agora é só criar dashboards e aproveitar o monitoramento 🔥

Se quiser posso te entregar um dashboard custom pronto também!
