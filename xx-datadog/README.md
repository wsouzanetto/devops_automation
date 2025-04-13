# Lab: Monitorando AKS com Datadog (Free Tier)

Este lab mostra como criar um cluster AKS no Free Tier e configurar o agente do Datadog corretamente para coletar métricas, logs e visualizar tudo no painel da Datadog.

---

## Etapa 1 — Criar Resource Group no Azure
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

## Etapa 4 — Adicionar repositório Helm do Datadog
```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

---

## Etapa 5 — Criar Secret com API Key da Datadog
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

## Etapa 7 — Instalar o Datadog com Helm
```bash
helm upgrade --install datadog-agent datadog/datadog \
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

## Etapa 8 — Fazer o deploy da aplicação `java-api` com Datadog APM + Logs

Crie um arquivo `deployment.yaml` com o seguinte conteúdo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api
  labels:
    app: java-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      annotations:
        ad.datadoghq.com/java-api.logs: '[{"source":"java","service":"java-api"}]'
      labels:
        app: java-api
    spec:
      containers:
        - name: java-api
          image: iesodias/java-api:latest
          ports:
            - containerPort: 8081
          env:
            - name: DD_AGENT_HOST
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: DD_ENV
              value: dev
            - name: DD_SERVICE
              value: java-api
            - name: DD_VERSION
              value: 1.0.0
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
---
apiVersion: v1
kind: Service
metadata:
  name: java-api
  labels:
    app: java-api
spec:
  type: LoadBalancer
  selector:
    app: java-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8081
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: java-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: java-api
  minReplicas: 1
  maxReplicas: 3
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

## Etapa 9 — Aplicar o deployment
```bash
kubectl apply -f deployment.yaml
```

---

## Etapa 10 — Obter IP público da aplicação
```bash
kubectl get svc java-api
```
Acesse via browser ou curl:
```bash
curl http://<IP_PUBLICO>
```

---

## Etapa 11 — Visualizar no Datadog
- Vá em **APM > Services** e veja os traces da aplicação `java-api`
- Vá em **Logs > Explorer** e filtre por `source:java` ou `service:java-api`

---

🔥 Pronto! Agora você tem uma aplicação Java no AKS monitorada com APM, logs e autoscaling totalmente integrados no Datadog.
