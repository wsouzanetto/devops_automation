## Lab 1: Pipeline Básica com GitHub Actions

### Objetivo

Criar um workflow simples para validar o funcionamento do GitHub Actions em um repositório novo.

---

### 1. Criar estrutura local

```bash
mkdir workspace-devops-automation
cd workspace-devops-automation
```

---

### 2. Criar repositório no GitHub

1. Acesse [https://github.com](https://github.com)
2. Clique em “New Repository”
3. Nome: `workspace-devops-automation`
4. Deixe sem README, sem .gitignore (vamos configurar localmente)

---

### 🔗 3. Inicializar Git e conectar com repositório remoto

```bash
git init
git remote add origin https://github.com/SEU_USUARIO/workspace-devops-automation.git
echo "# Projeto DevOps Automation" > README.md
git add .
git commit -m "first commit"
git push -u origin main
```

---

### 4. Criar estrutura do GitHub Actions

```bash
mkdir -p .github/workflows
```

---

### 5. Criar o workflow básico

Crie o arquivo `.github/workflows/hello.yml`:

```yaml
name: Hello GitHub Actions

on:
  push:
    branches:
      - main

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Exibir mensagem no log
        run: echo "✅ GitHub Actions funcionando com sucesso!"
```

---

### 🚀 6. Subir para o GitHub

```bash
git add .
git commit -m "add basic GitHub Actions workflow"
git push
```

---

### 7. Verificar resultado

1. Acesse o repositório no GitHub
2. Clique na aba **Actions**
3. Verifique se a pipeline "Hello GitHub Actions" foi executada com sucesso

---

Pronto! Agora seu ambiente está configurado e o GitHub Actions está funcionando corretamente ✨

# Lab 1: CI/CD para Aplicação Java com Push no Azure Container Registry (ACR)

Este laboratório tem como objetivo clonar um projeto Java existente, configurar variáveis sensíveis, e executar uma pipeline de CI/CD no GitHub Actions que:

* Compila a aplicação com Maven
* Roda os testes
* Constrói a imagem Docker
* Realiza scan de segurança com Trivy
* Realiza login no Azure
* Faz push da imagem para o Azure Container Registry (ACR)

---

## ✨ Requisitos

* Conta no [Azure](https://portal.azure.com)
* Azure CLI instalado e autenticado
* Permissões para criar um **Azure Container Registry**
* Conta no [GitHub](https://github.com)
* Projeto GitHub com pipeline GitHub Actions habilitado

---

## 🏗️ Criar o Azure Container Registry (ACR)

Execute os comandos abaixo para criar um ACR básico:

```bash
# Variáveis
RESOURCE_GROUP="rg-devops-lab"
REGISTRY_NAME="devopsautomationid"
LOCATION="eastus"

# Criar grupo de recursos
az group create --name $RESOURCE_GROUP --location $LOCATION

# Criar o ACR
az acr create --resource-group $RESOURCE_GROUP \
  --name $REGISTRY_NAME \
  --sku Basic \
  --admin-enabled false
```

> 📝 O nome do ACR deve ser único globalmente e conter apenas letras minúsculas e números. Ele será acessado como `devopsautomationid.azurecr.io`.

---

## ♻️ Etapas do Lab

### 1. Clonar o projeto Java original

```bash
# Clone o repositório da aplicação Java em uma pasta temporária
git clone https://github.com/iesodias/devops-automation-api.git temp-folder
cd temp-folder

# Copie os arquivos essenciais para o repositório do pipeline
cp -r pom.xml src Dockerfile ../workspace-devops-automation/
```

### 2. Criar secrets no GitHub

No repositório `workspace-devops-automation`, acesse:

**Settings > Secrets and variables > Actions > New repository secret**

Crie os seguintes secrets:

| Nome                  | Valor                                                                 |
|-----------------------|-----------------------------------------------------------------------|
| `AZURE_CREDENTIALS`   | JSON gerado com `az ad sp create-for-rbac --name "mdcgithubactions" --role contributor --scopes /subscriptions/SUA_SUBSCRIPTION --sdk-auth`                |
| `AZURE_REGISTRY_NAME` | Nome completo do seu ACR (ex: `devopsautomationid.azurecr.io`)       |

> ✅ O JSON de `AZURE_CREDENTIALS` deve incluir `clientId`, `clientSecret`, `tenantId`, etc.

---

### 3. Workflow CI/CD (.github/workflows/pipeline.yml)

Crie ou substitua o arquivo `pipeline.yml` no diretório `.github/workflows/` com o conteúdo abaixo:

```yaml
name: 🚀 Java API CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

env:
  ARTIFACT_NAME: "api-${{ github.sha }}.jar"
  IMAGE_NAME: "java-api:${{ github.sha }}"
  REGISTRY_NAME: ${{ secrets.AZURE_REGISTRY_NAME }}
  REGISTRY_IMAGE: "${{ secrets.AZURE_REGISTRY_NAME }}/java-api:${{ github.sha }}"

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: 🔍 Checkout Code
        uses: actions/checkout@v3

      - name: ☕ Setup Java
        uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: 📦 Build with Maven
        run: mvn clean package -DskipTests

      - name: 🧪 Run Tests
        run: mvn test

      - name: 🛠 Build Docker Image
        run: docker build -t ${{ env.IMAGE_NAME }} .

      - name: 🔐 Trivy Scan (Image)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.IMAGE_NAME }}
          format: table
          exit-code: 0
          severity: HIGH,CRITICAL

      - name: 🔐 Azure Login
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: 🔐 Docker Login to ACR
        run: az acr login --name ${{ env.REGISTRY_NAME }}

      - name: 🏷️ Tag Docker Image
        run: docker tag ${{ env.IMAGE_NAME }} ${{ env.REGISTRY_IMAGE }}

      - name: 📤 Push to Azure Container Registry
        run: docker push ${{ env.REGISTRY_IMAGE }}

      - name: 📦 Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: java-api-artifact
          path: target/*.jar

  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - name: 📥 Download JAR
        uses: actions/download-artifact@v4
        with:
          name: java-api-artifact

      - name: 🚀 Fake Deploy
        run: |
          echo "✅ Build: ${{ env.ARTIFACT_NAME }}"
          echo "📡 Fake deploy to: https://staging.example.com"
````

> ✅ Este workflow compila, testa, escaneia, faz push no ACR e simula um deploy para staging.

---

### 4. Executar o pipeline

Dado que o projeto está atualizado e com os secrets criados:

```bash
git add .
git commit -m "setup pipeline ci/cd"
git push origin main
```

O GitHub Actions deve iniciar automaticamente a execução do workflow.

---

### ⚡ Validação

* Acesse o GitHub > Actions e verifique a execução da pipeline
* A imagem deve estar publicada em seu ACR
* Os testes devem rodar com sucesso
* O scan do Trivy deve mostrar eventuais vulnerabilidades, mas sem falhar
* O deploy "fake" deve aparecer no log


# Lab – Infraestrutura com Terraform + Ansible + GitHub Actions (Destroy opcional)

Este lab ensina a criar uma VM no Azure com Terraform, configurar com Ansible e orquestrar tudo com GitHub Actions — incluindo um botão de destruição opcional.

---

## 🔧 O que você vai aprender

- Criar infraestrutura com Terraform
- Configurar VM com Ansible
- Usar GitHub Actions com `workflow_dispatch`
- Rodar `destroy` manualmente com botão

---

## 📁 Estrutura do Projeto

```
azure-vm-terraform-ansible/
├── infra/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── backend.tf (opcional)
├── ansible/
│   └── playbook.yml
├── .github/
│   └── workflows/
│       └── deploy.yml
└── README.md
```

---

## ✅ Pré-requisitos

- Azure CLI instalado e autenticado
- Conta no Azure com permissão
- Repositório no GitHub com Actions habilitado
- Secrets criadas:
  - `AZURE_CREDENTIALS`
  - `ARM_SUBSCRIPTION_ID`
  - `ADMIN_PASSWORD`

---

## 🚀 Etapas do Lab

### 1. Criar a estrutura do projeto

Execute o comando abaixo para criar as pastas e arquivos necessários:

```bash
mkdir -p .github/workflows infra ansible && touch .github/workflows/deploy.yml infra/{main.tf,variables.tf,outputs.tf,terraform.tfvars} ansible/playbook.yml README.md
```

### 2. Acesse a pasta do projeto

```bash
cd workspace-devops-automation
```

---

### 3. Configure o backend remoto (opcional)

```bash
RESOURCE_GROUP="rg-tfstate"
STORAGE_ACCOUNT="tfstatecurso$RANDOM"
CONTAINER_NAME="tfstate"
LOCATION="eastus"

az group create --name $RESOURCE_GROUP --location $LOCATION

az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --encryption-services blob

az storage container create \
  --name $CONTAINER_NAME \
  --account-name $STORAGE_ACCOUNT \
  --auth-mode login
```

---

### 4. Escreva o código Terraform

Use os arquivos `main.tf`, `variables.tf`, `outputs.tf` na pasta `infra` para definir a VM, NSG, IP, etc.

main.tf
```t
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-automation"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "subnet-automation"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "public_ip" {
  name                = "public-ip-vm"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Basic"
}


resource "azurerm_network_interface" "nic" {
  name                = "nic-vm"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.public_ip.id
  }
}

resource "azurerm_network_security_group" "nsg" {
  name                = "nsg-vm"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  security_rule {
    name                       = "SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "Swagger"
    priority                   = 1002
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "8081"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}


resource "azurerm_network_interface_security_group_association" "nsg_assoc" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                            = "vm-automation"
  resource_group_name             = azurerm_resource_group.rg.name
  location                        = azurerm_resource_group.rg.location
  size                            = "Standard_B1s"
  admin_username                  = "azureuser"
  admin_password                  = var.admin_password
  disable_password_authentication = false

  network_interface_ids = [
    azurerm_network_interface.nic.id
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts"
    version   = "latest"
  }
}

```

variables.tf
```t
variable "resource_group_name" {
  description = "Nome do Resource Group"
  type        = string
  default     = "rg-vm-automation"
}

variable "location" {
  description = "Região do Azure"
  type        = string
  default     = "East US"
}

variable "admin_password" {
  description = "Senha do usuário administrador"
  type        = string
  sensitive   = true
  default     = "Torresmo!@123!(*@)"
}

```

backend.tf
```t
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "tfstatecurso10998"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}

```

outputs.tf
```t
output "public_ip_address" {
  value       = azurerm_public_ip.public_ip.ip_address
  description = "Endereço IP público da VM"
}


output "nsg_name" {
  value = azurerm_network_security_group.nsg.name
}

output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}

```

---

### 5. Crie o playbook Ansible

Em `ansible/playbook.yml`, coloque o que deseja instalar (ex: Docker).


```yaml
---
- name: Instalar Docker no Ubuntu
  hosts: vm
  become: yes
  tasks:
    - name: Instalar dependências
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present
        update_cache: yes

    - name: Adicionar chave GPG oficial do Docker
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Adicionar repositório do Docker
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
        state: present

    - name: Atualizar cache do apt e instalar Docker
      apt:
        name: docker-ce
        state: present
        update_cache: yes

    - name: Adicionar usuário ao grupo Docker
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Iniciar e habilitar serviço Docker
      systemd:
        name: docker
        state: started
        enabled: yes
        
    - name: Executar container da aplicação Java
      docker_container:
        name: java-api
        image: iesodias/java-api:latest
        state: started
        restart_policy: always
        published_ports:
          - "8081:8081"
```

---

### 6. Configure o GitHub Actions

Em `.github/workflows/deploy.yml`, insira o workflow com `workflow_dispatch` e input `destroy`.

Você poderá rodar:

- `destroy: false` → cria infra + configura + testa
- `destroy: true` → apenas destrói infra


```yaml
name: Deploy Terraform to Azure

on:
  workflow_dispatch:
    inputs:
      destroy:
        description: 'Destruir infraestrutura?'
        required: true
        default: 'false'

permissions:
  id-token: write
  contents: read

jobs:
  validate:
    if: ${{ github.event.inputs.destroy == 'false' }}
    name: 🔎 Validar Terraform
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Instalar Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Validar Sintaxe e Plano
        working-directory: infra
        run: |
          export ARM_SUBSCRIPTION_ID=${{ secrets.ARM_SUBSCRIPTION_ID }}
          terraform init
          terraform fmt -check
          terraform validate
          terraform plan -out=tfplan

  deploy:
    if: ${{ github.event.inputs.destroy == 'false' }}
    name: 🚀 Deploy e Configuração
    needs: validate
    runs-on: ubuntu-latest
    outputs:
      vmName: ${{ steps.output_vm.outputs.vm_name }}
      adminUsername: ${{ steps.output_vm.outputs.admin_username }}
      publicIP: ${{ steps.output_vm.outputs.public_ip }}
      nsgName: ${{ steps.output_vm.outputs.nsg_name }}
      resourceGroup: ${{ steps.output_vm.outputs.resource_group }}
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Instalar Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Aplicar Terraform
        working-directory: infra
        run: |
          export ARM_SUBSCRIPTION_ID=${{ secrets.ARM_SUBSCRIPTION_ID }}
          terraform init
          terraform apply -auto-approve

      - name: Capturar Outputs
        id: output_vm
        working-directory: infra
        run: |
          echo "vm_name=vm-automation" >> $GITHUB_OUTPUT
          echo "admin_username=azureuser" >> $GITHUB_OUTPUT
          echo "public_ip=$(terraform output -raw public_ip_address)" >> $GITHUB_OUTPUT
          echo "nsg_name=$(terraform output -raw nsg_name)" >> $GITHUB_OUTPUT
          echo "resource_group=$(terraform output -raw resource_group_name)" >> $GITHUB_OUTPUT

      - name: Definir Variáveis de Ambiente
        run: |
          echo "VM_NAME=vm-automation" >> $GITHUB_ENV
          echo "ADMIN_USERNAME=azureuser" >> $GITHUB_ENV
          echo "PUBLIC_IP=${{ steps.output_vm.outputs.public_ip }}" >> $GITHUB_ENV
          echo "SSH_COMMAND=ssh azureuser@${{ steps.output_vm.outputs.public_ip }}" >> $GITHUB_ENV

      - name: Instalar Ansible e sshpass
        run: |
          sudo apt-get update
          sudo apt-get install -y ansible sshpass

      - name: Criar Inventário Ansible
        run: |
          echo "[vm]" > inventory
          echo "${{ env.PUBLIC_IP }} ansible_user=${{ env.ADMIN_USERNAME }} ansible_password=${{ secrets.ADMIN_PASSWORD }} ansible_ssh_common_args='-o StrictHostKeyChecking=no'" >> inventory

      - name: Executar Playbook Ansible
        run: |
          ansible-playbook -i inventory ansible/playbook.yml --extra-vars "ansible_sudo_pass=${{ secrets.ADMIN_PASSWORD }}"

  post-tests:
    if: ${{ github.event.inputs.destroy == 'false' }}
    name: ✅ Pós-Testes de Infra
    needs: deploy
    runs-on: ubuntu-latest
    env:
      PUBLIC_IP: ${{ needs.deploy.outputs.publicIP }}
      VM_NAME: ${{ needs.deploy.outputs.vmName }}
      NSG_NAME: ${{ needs.deploy.outputs.nsgName }}
      RESOURCE_GROUP: ${{ needs.deploy.outputs.resourceGroup }}
    steps:
      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Testar Swagger na Porta 8081
        run: |
          echo "Aguardando aplicação subir com Swagger..."
          sleep 30
          response=$(curl -s -o /dev/null -w "%{http_code}" http://$PUBLIC_IP:8081/swagger-ui/index.html)
          if [ "$response" != "200" ]; then
            echo "❌ Swagger não respondeu como esperado. Status HTTP: $response"
            exit 1
          else
            echo "✅ Swagger disponível em /swagger-ui/index.html na porta 8081!"
          fi

      - name: Verificar status da VM
        run: |
          status=$(az vm get-instance-view \
            --name "$VM_NAME" \
            --resource-group "$RESOURCE_GROUP" \
            --query "instanceView.statuses[?code=='PowerState/running'].displayStatus" \
            --output tsv)

          echo "Status da VM: $status"

          if [ "$status" != "VM running" ]; then
              echo "❌ A VM não está em execução!"
              exit 1
          else
              echo "✅ VM está rodando com sucesso!"
          fi

      - name: Verificar regra da NSG para porta 8081
        run: |
          result=$(az network nsg rule list \
            --nsg-name "$NSG_NAME" \
            --resource-group "$RESOURCE_GROUP" \
            --query "[?destinationPortRange=='8081' && access=='Allow']")

          if [ "$result" = "[]" ]; then
            echo "❌ Porta 8081 não está liberada na NSG!"
            exit 1
          else
            echo "✅ Porta 8081 está liberada corretamente na NSG!"
          fi

  destroy:
    if: ${{ github.event.inputs.destroy == 'true' }}
    name: 🧨 Destruir Infraestrutura
    runs-on: ubuntu-latest
    env:
      ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Login no Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Instalar Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Executar Terraform Destroy
        working-directory: infra
        run: |
          terraform init
          terraform destroy -auto-approve

```

### 7. Faça commit e push

```bash
git add .
git commit -m "Lab - Terraform + Ansible com botão de Destroy"
git push origin main
```

## ☁️ Rodando o Lab

1. Vá até a aba **Actions** do GitHub
2. Escolha o workflow `Deploy Terraform to Azure`
3. Clique em **Run workflow**
4. Selecione `destroy = false` para criar ou `true` para destruir

## ✅ Resultado Esperado

- Infra criada automaticamente
- Configuração da VM com Ansible
- Swagger disponível na porta 8081
- Destruição simples e segura via botão do GitHub

## 🧹 Dica Final

Deixe o valor padrão do `destroy` como `false` e oriente sua equipe a mudar para `true` **somente se quiser destruir tudo.**

---
