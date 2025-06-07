# 🧪 Lab - Instalando o Agente do Datadog em uma Instância EC2 na AWS

## 🎯 Objetivo
Realizar o provisionamento de uma instância EC2 na AWS e instalar o agente do Datadog para coletar métricas básicas de infraestrutura.

---

## ✅ Pré-requisitos
- Conta na AWS com permissões de EC2
- AWS CLI configurado (`aws configure`)
- Par de chaves criado (`devops-automation.pem`)
- Conta no Datadog com plano trial ativo
- Integração AWS conectada no portal do Datadog (Integrations > AWS)

---

## 🚀 Etapa 1 - Criar Security Group com portas necessárias

```bash
aws ec2 create-security-group \
  --group-name datadog-sg \
  --description "Security group para agente do Datadog"

aws ec2 authorize-security-group-ingress \
  --group-name datadog-sg \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-name datadog-sg \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

---

## 🖥️ Etapa 2 - Criar uma instância EC2 Amazon Linux 2

```bash
aws ec2 run-instances \
  --image-id ami-0c101f26f147fa7fd \
  --instance-type t2.micro \
  --key-name devops-automation \
  --security-groups datadog-sg \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=datadog-ec2}]' \
  --count 1
```

---

## 🔍 Etapa 3 - Obter o IP público da instância

```bash
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=datadog-ec2" \
--query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]" \
--output table
```

---

## 🔐 Etapa 4 - Acessar a instância via SSH

```bash
chmod 400 devops-automation.pem
ssh -i "devops-automation.pem" ec2-user@SEU_IP_PUBLICO
```

---

## 🐶 Etapa 5 - Instalar o agente do Datadog

1. Copie sua API Key no portal: `Integrations > APIs`

2. Rode na instância EC2:

```bash
DD_API_KEY="SUA_API_KEY" \
DD_SITE="datadoghq.com" \
bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

---

## 📈 Etapa 6 - Validar o status do agente

```bash
sudo datadog-agent status
```

Você deve ver diversas métricas coletadas, como CPU, memória, disco, etc.

---

## 🧭 Etapa 7 - Verificar no portal do Datadog

1. Acesse: https://app.datadoghq.com/
2. Vá até **Infrastructure > Host Map**
3. Confirme se o host aparece pelo nome ou ID
4. Visualize dashboards prontos como **EC2 Overview** ou **Host Overview**

---

## 🧹 Etapa 8 - Limpar o ambiente (opcional)

```bash
# Listar instâncias e capturar Instance ID
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datadog-ec2" \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text

# Terminar a instância
aws ec2 terminate-instances --instance-ids i-xxxxxxxxxxxxxxxxx

# Deletar o Security Group
aws ec2 delete-security-group --group-name datadog-sg
```

---

## ✅ Pronto
Sua EC2 foi configurada com o agente do Datadog e a integração com AWS está funcionando.
Você pode agora criar dashboards, monitores e explorar recursos de observabilidade de forma completa.


