# Lab AWS - Criando Instância EC2 (Ubuntu) via Console

## Objetivo  
Criar uma instância EC2 com Ubuntu na AWS, configurar acesso SSH e segurança básica.

 

## Pré-requisitos  
- Conta AWS ativa  
- Acesso ao [Console AWS](https://console.aws.amazon.com)  

 

## Passo a Passo

> 🆕 Este lab inclui: IP público fixo (Elastic IP) e script de user-data para inicialização da VM com Docker e Nginx já instalados.  

### 1. Acessar o Console AWS  
1. Faça login no [Console AWS](https://console.aws.amazon.com)  
2. Na barra de busca, digite `EC2` e selecione o serviço  

### 2. Iniciar Criação da Instância  
1. Clique em `"Executar instância"` (botão laranja)  

### 3. Configuração Básica  
```plaintext
Nome da instância: devops-aws-ec2  
Sistema operacional: Ubuntu Server 22.04 LTS (HVM)  
Tipo de instância: t2.micro (Free Tier elegível)  
Par de chaves: Selecione existente ou crie novo (.pem)  
```

### 4. Configuração de Rede  
```plaintext
Rede VPC: vpc-default (ou crie nova)  
Sub-rede: Escolha uma zona de disponibilidade  
Grupo de segurança:  
  - Nome: devops-aws-sg  
  - Regras de entrada:  
    * SSH (22) - Meu IP  
    * HTTP (80) - Opcional  
```

### 5. Armazenamento  
```plaintext
Tipo de volume: gp3 (SSD)  
Tamanho: 8 GB (Free Tier)  
```

### 6. Implantação
```plaintext
Clique em "Executar instância"
Aguarde status "Running" (~2 minutos)
```

 

## 🔹 Configurar Elastic IP (IP Público fixo)

### 1. Alocar Elastic IP
1. No Console AWS, vá até "Elastic IPs"
2. Clique em "Alocar Elastic IP"
3. Confirme a alocação

### 2. Associar Elastic IP à instância
1. Selecione o IP criado
2. Clique em "Ações > Associar"
3. Escolha a instância criada (devops-aws-ec2)

Após isso, o IP será fixo mesmo que a instância seja reiniciada.

 

## 🔹 Script de User Data (instalação automática)

Durante a criação da instância (na aba "Configurar instância"), adicione o seguinte script no campo **"Script de dados do usuário"**:

```bash
#!/bin/bash
apt update -y
apt upgrade -y
apt install -y docker.io nginx
systemctl enable docker
systemctl start nginx
```

Esse script será executado automaticamente na primeira inicialização, instalando Docker, Nginx e ativando o serviço.

 

## Conexão via SSH

### 1. Localizar IP Público  
No console EC2 > Instâncias > copie o IPv4 Público

### 2. Conectar (Linux/macOS)
```bash
chmod 400 chave-devops.pem  # Dar permissão à chave
ssh -i "chave-devops.pem" ubuntu@<IP_PUBLICO>
```

### 3. Conectar (Windows)  
Use o PuTTY ou Windows Terminal  
Converta a chave .pem para .ppk (via PuTTYgen)  

Conecte com:
```bash
ssh -i "chave-devops.ppk" ubuntu@<IP_PUBLICO>
```

 

## Comandos Úteis (Pós-Instalação)

### Atualizar pacotes
```bash
sudo apt update && sudo apt upgrade -y
```

### Instalar ferramentas básicas
```bash
sudo apt install -y docker.io nginx
```

 

## Gerenciamento via AWS CLI

### Listar instâncias
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId, State:State.Name, IP:PublicIpAddress}' --output table
```

### Parar instância
```bash
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

### Terminar instância
```bash
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

## Boas Práticas

Nunca use regras de segurança com `0.0.0.0/0` para SSH  
💸 Configure alertas de billing para evitar custos inesperados  

## Diferenciais desta versão:
- Foco em Ubuntu + Free Tier
- Comandos para Windows/Linux/macOS
- Explicação de segurança crítica
- Formatação 100% compatível com Markdown

 