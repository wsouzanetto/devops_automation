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

Quer que eu adicione algum recurso específico junto com o Resource Group? (Storage Account, VNET, etc.) 😊