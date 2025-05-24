# Lab 1 – Criando um Resource Group com Terraform (Azure)

## 🎯 Objetivo
Criar um Resource Group na Azure usando Terraform, passo a passo.

## 📁 Estrutura de Diretórios
```bash
mkdir -p ~/labs/terraform/lab1-resource-group
cd ~/labs/terraform/lab1-resource-group
```

## 📄 Passo 1 – Criar o arquivo de provider
```bash
vi provider.tf
```
Conteúdo:
```hcl
provider "azurerm" {
  features {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.0"
}
```

## 📄 Passo 2 – Criar o arquivo de configuração principal
```bash
vi main.tf
```
Conteúdo:
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "mdc-rg"
  location = "East US"
}
```

## ✅ Passo 3 – Inicializar o Terraform
```bash
terraform init
```

## 🔍 Passo 4 – Validar e revisar o plano
```bash
terraform validate
terraform plan
```

## 🚀 Passo 5 – Aplicar o código e provisionar
```bash
terraform apply
```
Confirme com `yes` quando solicitado.

## 🧼 Passo 6 – Destruir os recursos (opcional)
```bash
terraform destroy
```

---

# Lab 2 – Criando uma VM Ubuntu com Variáveis e Data Source (Azure)

## 🎯 Objetivo
Criar uma máquina virtual Ubuntu na Azure usando Terraform com variáveis e buscando o resource group existente via `data`.

## 📁 Estrutura de Diretórios
```bash
mkdir -p ~/labs/terraform/lab3-vm-variaveis
cd ~/labs/terraform/lab3-vm-variaveis
```

## 📄 Passo 1 – Criar o arquivo de provider
```bash
vi provider.tf
```
Conteúdo:
```t
provider "azurerm" {
  features {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.0"
}
```

## 📄 Passo 2 – Definir variáveis
```t
vi variables.tf
```
Conteúdo:
```hcl
variable "location" {
  default = "East US"
}

variable "resource_group_name" {
  default = "mdc-rg"
}

variable "admin_username" {
  default = "azureuser"
}

variable "admin_password" {
  default = "SenhaForte123!@#"
}
```

## 📄 Passo 3 – Criar o arquivo de configuração principal
```bash
vi main.tf
```
Conteúdo:
```t
data "azurerm_resource_group" "rg" {
  name = var.resource_group_name
}

resource "azurerm_public_ip" "public_ip" {
  name                = "mdc-public-ip"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Basic"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "mdc-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "mdc-subnet"
  resource_group_name  = data.azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "nic" {
  name                = "mdc-nic"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.public_ip.id
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "mdc-vm"
  resource_group_name = data.azurerm_resource_group.rg.name
  location            = var.location
  size                = "Standard_B1s"
  admin_username      = var.admin_username
  admin_password      = var.admin_password
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

resource "azurerm_network_security_group" "nsg" {
  name                = "mdc-nsg"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "nic_nsg" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}
```

## ✅ Passo 4 – Inicializar o Terraform
```bash
terraform init
```

## 🔍 Passo 5 – Validar e revisar o plano
```bash
terraform validate
terraform plan
```

## 🚀 Passo 6 – Aplicar o código e provisionar
```bash
terraform apply
```
Confirme com `yes` quando solicitado.

## 🧼 Passo 7 – Destruir os recursos (opcional)
```bash
terraform destroy
```

---
Esse lab aproveita um resource group existente e usa variáveis pra tornar o código mais flexível e fácil de reaproveitar. No próximo lab podemos separar a rede em um módulo reutilizável.