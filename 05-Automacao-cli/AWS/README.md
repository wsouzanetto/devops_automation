# Lab AWS – Criando e Manipulando S3 com AWS CLI

Este laboratório ensina como criar e manipular buckets no Amazon S3 utilizando apenas a AWS CLI, sem interface gráfica. Ideal para quem está aprendendo automação e linha de comando na nuvem.

---

## Pré-requisitos

* AWS CLI instalada e configurada (`aws configure`)
* Permissões de acesso ao S3

---

## 1. Criar um bucket S3

```bash
aws s3api create-bucket --bucket meu-lab-s3 --region us-east-1
```

> Cria um bucket chamado `meu-lab-s3` na região `us-east-1`.
> Atenção: o nome do bucket deve ser único globalmente.

---

## 2. Listar todos os buckets S3 da conta

```bash
aws s3api list-buckets --query "Buckets[].Name"
```

> Lista os nomes de todos os buckets criados na sua conta AWS.

---

## 3. Enviar um arquivo para o bucket

```bash
echo "teste" > arquivo.txt
aws s3 cp arquivo.txt s3://meu-lab-s3/
```

> Cria um arquivo local e envia para o bucket.

---

## 4. Listar arquivos dentro do bucket

```bash
aws s3 ls s3://meu-lab-s3/
```

> Mostra todos os objetos salvos no bucket.

---

## 5. Baixar um arquivo do bucket

```bash
aws s3 cp s3://meu-lab-s3/arquivo.txt arquivo-download.txt
```

> Baixa um objeto específico do bucket para o seu computador.

---

## 6. Criar uma pasta (prefixo) no bucket

```bash
aws s3api put-object --bucket meu-lab-s3 --key pasta1/
```

> Cria uma pasta lógica no bucket.
> No S3,
## Lab 10 – Criando e Manipulando VMs na AWS via CLI

### 🎙️ Fala inicial

Nesse laboratório vamos aprender a criar uma máquina virtual na AWS usando apenas a linha de comando
Vamos usar a VPC padrão e variáveis para automatizar cada passo do processo
Esse tipo de automação é essencial para quem quer integrar criação de infraestrutura em pipelines DevOps

---

### 1. Objetivo

Criar e gerenciar instâncias EC2 na AWS usando a linha de comando, aproveitando a VPC e subnets padrão, com uso de variáveis para reaproveitamento.

---

### 2. Pré-requisitos

* AWS CLI instalada e configurada (`aws configure`)
* Permissões para EC2, VPC e IAM

---

### 3. Variáveis iniciais do ambiente

```bash
VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" \
  --output text)
```

> `--filters Name=isDefault,Values=true`: filtra as VPCs marcadas como padrão na conta
> `--query "Vpcs[0].VpcId"`: retorna o ID da primeira VPC padrão
> `--output text`: remove formatação JSON, retornando apenas o texto puro

```bash
SUBNET_ID=$(aws ec2 describe-subnets \
  --filters Name=default-for-az,Values=true \
  --query "Subnets[0].SubnetId" \
  --output text)
```

> `--filters Name=default-for-az,Values=true`: filtra subnets padrão da zona de disponibilidade
> `--query "Subnets[0].SubnetId"`: retorna o ID da primeira subnet disponível

```bash
KEY_NAME="devops-keypair"
SG_NAME="devops-sg-ie"
```

> Define nomes de referência para a chave SSH e grupo de segurança

---

### 4. Criar par de chaves para acesso

```bash
aws ec2 create-key-pair \
  --key-name "$KEY_NAME" \
  --query 'KeyMaterial' \
  --output text > "$KEY_NAME.pem"
```

> `--key-name`: nome da chave a ser criada
> `--query 'KeyMaterial'`: extrai apenas o conteúdo da chave privada
> `--output text`: evita JSON
> `> "$KEY_NAME.pem"`: salva o conteúdo no arquivo PEM local

```bash
chmod 400 "$KEY_NAME.pem"
```

> Define permissão segura para o arquivo (leitura apenas pelo dono)

---

### 5. Criar security group

```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name "$SG_NAME" \
  --description "Acesso SSH" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' \
  --output text)
```

> Cria um grupo de segurança e captura seu ID com a opção `--query 'GroupId'`

---

### 6. Liberar acesso SSH

```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

> `--group-id`: ID do grupo que será alterado
> `--protocol tcp`: define o protocolo
> `--port 22`: libera a porta 22 (SSH)
> `--cidr 0.0.0.0/0`: permite acesso de qualquer IP (não recomendado para produção)

---

### 7. Obter ID de uma AMI Amazon Linux 2

```bash
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text)
```

> `--owners amazon`: filtra apenas AMIs oficiais da Amazon
> `--filters`: filtra AMIs Amazon Linux 2 disponíveis
> `sort_by(...) | [-1]`: ordena pela data de criação e pega a mais recente
> `--output text`: retorna apenas o ID

---

### 8. Criar instância EC2

```bash
INSTANCE_INFO=$(aws ec2 run-instances \
  --image-id "$AMI_ID" \
  --count 1 \
  --instance-type t2.micro \
  --key-name "$KEY_NAME" \
  --security-group-ids "$SG_ID" \
  --subnet-id "$SUBNET_ID" \
  --associate-public-ip-address \
  --query 'Instances[0].[InstanceId, PublicIpAddress]' \
  --output text)
```

> Cria a instância com IP público e captura ID e IP via query

```bash
INSTANCE_ID=$(echo "$INSTANCE_INFO" | awk '{print $1}')
PUBLIC_IP=$(echo "$INSTANCE_INFO" | awk '{print $2}')
```

> Extrai valores separados da resposta

```bash
echo "Instância criada: $INSTANCE_ID"
echo "IP Público: $PUBLIC_IP"
```

> Exibe os dados no terminal

---

### 9. Conectar via SSH

```bash
ssh -i "$KEY_NAME.pem" ec2-user@"$PUBLIC_IP"
```

> Realiza a conexão com a instância EC2 utilizando a chave privada

---

### 10. Parar a instância

```bash
aws ec2 stop-instances --instance-ids "$INSTANCE_ID"
```

> Envia comando de parada para a instância

---

### 11. Iniciar novamente

```bash
aws ec2 start-instances --instance-ids "$INSTANCE_ID"
```

> Inicia a instância novamente

---

### 12. Terminar e limpar recursos

```bash
aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
```

> Deleta a instância EC2

```bash
aws ec2 delete-security-group --group-id "$SG_ID"
```

> Remove o grupo de segurança criado

```bash
rm "$KEY_NAME.pem"
```

> Remove a chave local do sistema

---


## Lab 11 – Criando VMs em Lote com Script Bash na AWS CLI

### 1. Objetivo

Automatizar a criação de múltiplas instâncias EC2 usando um script em shell com loops e variáveis.

---

### 2. Pré-requisitos

* AWS CLI configurada (`aws configure`)
* Permissões de EC2, VPC, subnets e security groups

---

### 3. Criar diretório do projeto

```bash
mkdir -p ~/labs/aws/vm-lote # CASO TENHA APAGADO
cd ~/labs/aws/vm-lote
touch criar-vms-aws.sh
chmod +x criar-vms-aws.sh
```

---

### 4. Conteúdo do script `criar-vms.sh`

```bash
## Lab 11 – Criando VMs em Lote com Script Bash na AWS CLI

### 1. Objetivo

Automatizar a criação de múltiplas instâncias EC2 usando um script em shell com loops e variáveis.

---

### 2. Pré-requisitos

* AWS CLI configurada (`aws configure`)
* Permissões de EC2, VPC, subnets e security groups

---

### 3. Criar diretório do projeto

```bash
mkdir -p ~/labs/aws/vm-lote
cd ~/labs/aws/vm-lote
touch criar-vms.sh
chmod +x criar-vms.sh
```

---

### 4. Conteúdo do script `criar-vms.sh`

```bash
#!/bin/bash

KEY_NAME="devops-keypair"
SG_NAME="devops-sg-ie"

VPC_ID=$(aws ec2 describe-vpcs \
  --filters Name=isDefault,Values=true \
  --query "Vpcs[0].VpcId" --output text)

SUBNET_ID=$(aws ec2 describe-subnets \
  --filters Name=default-for-az,Values=true \
  --query "Subnets[0].SubnetId" --output text)

AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text)

# Verifica se a keypair já existe na AWS
aws ec2 describe-key-pairs --key-names "$KEY_NAME" >/dev/null 2>&1

if [ $? -ne 0 ]; then
  echo "Criando keypair $KEY_NAME"
  aws ec2 create-key-pair --key-name "$KEY_NAME" \
    --query 'KeyMaterial' --output text > "$KEY_NAME.pem"
  chmod 400 "$KEY_NAME.pem"
else
  echo "Keypair $KEY_NAME já existe na AWS. Pulando criação."
  if [ ! -f "$KEY_NAME.pem" ]; then
    echo "⚠️ Arquivo local $KEY_NAME.pem não existe. Crie manualmente ou baixe da criação original."
    exit 1
  fi
fi

# Criar security group
SG_ID=$(aws ec2 create-security-group \
  --group-name "$SG_NAME" \
  --description "Acesso via loop" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)

# Liberar porta 22
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" --protocol tcp --port 22 --cidr 0.0.0.0/0

# Lista de nomes
VMS=(vm01 vm02 vm03)

for NAME in "${VMS[@]}"; do
  echo "Criando instância: $NAME"

  INSTANCE_ID=$(aws ec2 run-instances \
    --image-id "$AMI_ID" \
    --count 1 \
    --instance-type t2.micro \
    --key-name "$KEY_NAME" \
    --security-group-ids "$SG_ID" \
    --subnet-id "$SUBNET_ID" \
    --associate-public-ip-address \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$NAME}]" \
    --query 'Instances[0].InstanceId' --output text)

  echo "$NAME criada com ID: $INSTANCE_ID"

  # Aguardar a instância estar pronta
  aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"
  echo "$NAME está em execução"
done
```

---

### 5. Executar o script

```bash
./criar-vms.sh
```

---

### 6. Validar as instâncias

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=vm01,vm02,vm03" \
  --query "Reservations[].Instances[].PublicIpAddress" --output text
```

---

### 7. Fala final

Muito bom com esse script conseguimos criar várias VMs com nome personalizado
Isso mostra como loops e variáveis ajudam na automação em larga escala
Te vejo na próxima aula

```

---

### 5. Executar o script

```bash
./criar-vms.sh
```

---

### 6. Validar as instâncias

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=vm01,vm02,vm03" \
  --query "Reservations[].Instances[].PublicIpAddress" --output text
```

---

## Lab 12 – Script para Destruir VMs e Recursos na AWS via CLI

### 1. Objetivo

Remover instâncias EC2 criadas em lote e também os recursos associados como security group e chave de acesso

---

### 2. Estrutura esperada

Este script assume que as VMs foram criadas com nomes como `vm01`, `vm02`, `vm03`, e que os recursos foram nomeados da seguinte forma:

```bash
KEY_NAME="devops-keypair"
SG_NAME="devops-sg-ie"
```

---

### 3. Criar o script de destruição

```bash
mkdir -p ~/labs/aws/vm-lote
cd ~/labs/aws/vm-lote
touch destruir-vms.sh
chmod +x destruir-vms.sh
```

---

### 4. Conteúdo do `destruir-vms.sh`

```bash
#!/bin/bash

export AWS_PAGER=""

KEY_NAME="devops-keypair-01"
SG_NAME="devops-sg-ie"

# Lista de nomes criados
VMS=(vm01 vm02 vm03)

# Identificar os Instance IDs
INSTANCE_IDS=()

for NAME in "${VMS[@]}"; do
  INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=$NAME" "Name=instance-state-name,Values=running,stopped" \
    --query "Reservations[*].Instances[*].InstanceId" --output text)

  if [ -n "$INSTANCE_ID" ]; then
    echo "Finalizando $NAME com ID $INSTANCE_ID"
    aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
    INSTANCE_IDS+=($INSTANCE_ID)
  else
    echo "Instância $NAME não encontrada ou já terminada"
  fi
done

# Aguardar finalização de TODAS as instâncias coletadas
if [ ${#INSTANCE_IDS[@]} -gt 0 ]; then
  echo "Aguardando término das instâncias..."
  aws ec2 wait instance-terminated --instance-ids "${INSTANCE_IDS[@]}"
  echo "Instâncias finalizadas com sucesso"
else
  echo "Nenhuma instância para aguardar término"
fi

# Deletar security group com verificação
SG_ID=$(aws ec2 describe-security-groups \
  --group-names "$SG_NAME" \
  --query 'SecurityGroups[0].GroupId' --output text 2>/dev/null)

if [ -n "$SG_ID" ]; then
  echo "Aguardando liberação do Security Group..."
  sleep 10  # pequena pausa pra garantir liberação
  aws ec2 delete-security-group --group-id "$SG_ID"
  echo "Security group removido com sucesso"
else
  echo "Security group $SG_NAME não encontrado ou já removido"
fi

# Deletar chave remota e local
aws ec2 delete-key-pair --key-name "$KEY_NAME"
rm -f "$KEY_NAME.pem"
echo "Keypair e arquivo local removidos"

```

---

### 5. Executar o script

```bash
./destruir-vms.sh
```

---

### 6. Fala final

Feito com sucesso destruímos todas as VMs em lote e limpamos os recursos da AWS
Esse tipo de script é essencial para evitar custos e manter o ambiente limpo
Te vejo na próxima aula
