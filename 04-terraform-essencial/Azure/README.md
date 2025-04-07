# Lab Terraform - Criando Resource Group no Azure

## Objetivo
Criar um Resource Group no Azure usando Terraform (Infraestrutura como Código).

---

## Pré-requisitos
- [Azure CLI instalado](https://learn.microsoft.com/pt-br/cli/azure/install-azure-cli)
- [Terraform instalado](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
- Acesso a uma assinatura Azure

---

## Passo a Passo

### 1. Autenticar na Azure
```bash
az login
```

### 2. Criar diretório do projeto
```bash
mkdir terraform-azure-rg && cd terraform-azure-rg
```

### 3. Criar arquivo main.tf
```hcl
# Configure o provider Azure
provider "azurerm" {
  features {}
}

# Crie um Resource Group
resource "azurerm_resource_group" "devops_rg" {
  name     = "devops-resources-rg"
  location = "Brazil South"

  tags = {
    Environment = "Dev"
    Team        = "DevOps"
  }
}
```

### 4. Criar arquivo versions.tf
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}
```

### 5. Inicializar Terraform
```bash
terraform init
```

Saída esperada:
```plaintext
Terraform initialized successfully!
```

### 6. Verificar plano de execução
```bash
terraform plan
```

Confirme que aparecerá:
```plaintext
Plan: 1 to add, 0 to change, 0 to destroy.
```

### 7. Aplicar a configuração
```bash
terraform apply
```

Digite `yes` quando solicitado.

### 8. Verificar criação (CLI)
```bash
az group show --name devops-resources-rg --output jsonc
```

---

## Comandos Úteis

### Listar Resource Groups via Terraform
```bash
terraform state list
```

### Destruir recursos
```bash
terraform destroy
```

### Limpar e validar sintaxe
```bash
terraform fmt && terraform validate
```

---

## Estrutura Final do Projeto
```plaintext
terraform-azure-rg/
├── main.tf            # Configuração principal
├── versions.tf        # Versões de providers
├── terraform.tfstate  # Estado atual (gerado automaticamente)
└── .terraform/        # Cache de plugins (gerado automaticamente)
```

---

## Melhores Práticas

- ✅ **Versionamento**: Commit seus arquivos `.tf` no Git
- ❌ **Segurança**: Nunca commit arquivos `.tfstate`
- 🛠️ **Variáveis**: Use `variables.tf` para parametrização (exemplo abaixo)

### Exemplo avançado: variables.tf
```hcl
variable "rg_name" {
  description = "Nome do Resource Group"
  default     = "devops-resources-rg"
}

variable "location" {
  description = "Região Azure"
  default     = "brazilsouth"
}
```

Atualize o `main.tf` para usar `var.rg_name` e `var.location`

---

## Como Customizar:
1. Para mudar o nome do Resource Group, edite o `main.tf`
2. Para adicionar mais recursos, inclua novos blocos `resource` após o Resource Group
3. Para usar outras regiões, consulte [Regiões Azure](https://azure.microsoft.com/pt-br/explore/global-infrastructure/geographies/#overview)

---

# Lab Azure Storage Account - Estrutura Profissional

## Estrutura de Arquivos
```plaintext
storage-lab/
├── main.tf          # Recursos principais
├── variables.tf     # Variáveis de entrada
├── outputs.tf       # Saídas do módulo
├── versions.tf      # Versões de providers
└── terraform.tfvars # Valores das variáveis
```

### 1. versions.tf
```hcl
terraform {
  required_version = ">= 1.3.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}
```

### 2. variables.tf
```hcl
variable "resource_group_name" {
  description = "Nome do Resource Group"
  type        = string
}

variable "location" {
  description = "Região Azure"
  type        = string
  default     = "brazilsouth"
}

variable "storage_account_name" {
  description = "Nome da Storage Account (3-24 chars, alfanumérico)"
  type        = string
}

variable "account_tier" {
  description = "Tier da Storage Account (Standard/Premium)"
  type        = string
  default     = "Standard"
}

variable "account_replication_type" {
  description = "Tipo de replicação (LRS/GRS/ZRS)"
  type        = string
  default     = "LRS"
}

variable "enable_https_traffic_only" {
  description = "Forçar tráfego HTTPS"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Tags para recursos"
  type        = map(string)
  default     = {}
}
```

### 3. terraform.tfvars
```hcl
resource_group_name       = "devops-storage-rg"
location                  = "brazilsouth"
storage_account_name      = "devopslabstorage123" # Substitua por um nome único
account_tier              = "Standard"
account_replication_type  = "GRS"
tags = {
  Environment = "dev"
  ManagedBy   = "Terraform"
}
```

### 4. main.tf
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "this" {
  name     = var.resource_group_name
  location = var.location
  tags     = var.tags
}

resource "azurerm_storage_account" "this" {
  name                     = lower(var.storage_account_name)
  resource_group_name      = azurerm_resource_group.this.name
  location                 = azurerm_resource_group.this.location
  account_tier             = var.account_tier
  account_replication_type = var.account_replication_type
  account_kind             = "StorageV2"

  enable_https_traffic_only = var.enable_https_traffic_only
  min_tls_version            = "TLS1_2"
  shared_access_key_enabled  = true

  blob_properties {
    versioning_enabled  = true
    change_feed_enabled = true

    container_delete_retention_policy {
      days = 30
    }
  }

  network_rules {
    default_action             = "Deny"
    ip_rules                   = ["100.0.0.0/16"] # Substitua pelos IPs permitidos
    virtual_network_subnet_ids = []
    bypass                     = ["AzureServices"]
  }

  tags = merge(var.tags, {
    StorageType = "GeneralPurposeV2"
  })
}

resource "azurerm_storage_container" "example" {
  name                  = "devops-container"
  storage_account_name  = azurerm_storage_account.this.name
  container_access_type = "private"
}
```

### 5. outputs.tf
```hcl
output "storage_account_id" {
  description = "ID da Storage Account"
  value       = azurerm_storage_account.this.id
}

output "primary_blob_endpoint" {
  description = "Endpoint primário para Blob Storage"
  value       = azurerm_storage_account.this.primary_blob_endpoint
}

output "primary_access_key" {
  description = "Chave de acesso primária"
  value       = azurerm_storage_account.this.primary_access_key
  sensitive   = true
}

output "connection_string" {
  description = "String de conexão"
  value       = azurerm_storage_account.this.primary_connection_string
  sensitive   = true
}
```

---

## Como Executar
```bash
# Inicializar providers
terraform init

# Verificar plano de execução
terraform plan

# Aplicar configuração
terraform apply

# Destruir recursos (quando necessário)
terraform destroy
```

---

## Features Avançadas Incluídas

### Segurança reforçada:
- TLS 1.2 obrigatório
- Network rules configuráveis
- HTTPS obrigatório

### Data Protection:
- Versionamento de blobs
- Change feed habilitado
- Retention policy (30 dias)

### Boas práticas:
- Tags padronizadas
- Outputs sensíveis marcados
- Validação de nomes

### Flexibilidade:
- Tipo de replicação configurável
- Tier (Standard/Premium) parametrizável

---

## Para Ambiente Production:

1. Crie um arquivo `production.tfvars` com:
```hcl
account_replication_type    = "GRS"
enable_https_traffic_only  = true
tags = {
  Environment = "production"
  Critical    = "true"
}
```

2. Aplique com:
```bash
terraform apply -var-file="production.tfvars"
```

