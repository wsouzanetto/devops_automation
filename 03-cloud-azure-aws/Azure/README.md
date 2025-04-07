# Lab 1 – Criando VM Ubuntu no Azure via Portal (DevOps)

## Objetivo
Criar uma VM Ubuntu 22.04 LTS no Azure via portal com configurações voltadas para DevOps.

---

## Passo a Passo

### 1. Acessar o Portal Azure
```plaintext
1. Acesse https://portal.azure.com
2. Clique em "Criar um recurso"
```

### 2. Selecionar Imagem Ubuntu
```plaintext
1. Busque por "Máquina Virtual"
2. Selecione:
   - Imagem: Ubuntu Server 22.04 LTS - Gen2
```

### 3. Configuração Básica
```plaintext
Grupo de recursos: devops-ubuntu-rg (novo)
Nome da VM: devops-ubuntu-vm
Região: Brazil South
Tamanho: Standard_B2s (2 vCPUs, 4 GiB RAM)
Autenticação: Senha
Usuário: devopsadmin
Senha: [Defina uma senha forte!]
```

### 4. Configuração de Rede
```plaintext
Rede virtual: devops-ubuntu-vnet (nova)
IP Público: devops-ubuntu-ip (novo)
Portas abertas: SSH (22)
```

### 5. Implantação
```plaintext
Clique em "Revisar + criar"
Após validação, clique em "Criar"
```

---

## Conexão via SSH

### Usando Azure CLI para obter o IP
```bash
ssh devopsadmin@$(az vm show -d -g devops-ubuntu-rg -n devops-ubuntu-vm --query publicIps -o tsv)
```

### Alternativa Manual
```bash
ssh devopsadmin@<IP_PUBLICO>
```

Obtenha o IP público da VM diretamente pelo portal se necessário.

---

## Comandos Azure CLI úteis

### Listar VMs
```bash
az vm list -g devops-ubuntu-rg -o table
```

### Parar a VM
```bash
az vm stop -g devops-ubuntu-rg -n devops-ubuntu-vm
```

### Deletar todos os recursos
```bash
az group delete -n devops-ubuntu-rg --yes
```

---

## Dicas de Segurança

- Use `ssh-keygen` e autenticação por chave pública sempre que possível
- Restrinja o NSG para aceitar conexões apenas do seu IP
- Sempre pare a VM quando não estiver em uso para evitar cobranças

---

Esse lab te ajuda a entender como provisionar e gerenciar uma VM no Azure com foco em DevOps. Podemos seguir com labs de automação usando CLI, Bicep ou Terraform se quiser.

