# ✅ Pré-requisitos para o Lab DevOps Automation

Este documento lista os pré-requisitos necessários para acompanhar todos os módulos do curso DevOps Automation. Siga o passo a passo abaixo para garantir que seu ambiente está pronto.

---

## 1. Sistema Operacional

Recomendado: **Linux Ubuntu 22.04 LTS** (pode ser WSL, VM ou nativo)

Verifique a versão:
```bash
lsb_release -a
```

### 💻 Caso você esteja usando Windows:

Recomenda-se usar o **WSL 2** com Ubuntu. Siga os passos abaixo:

#### 1.1 Ativar o WSL:
```powershell
wsl --install
```
Isso instala o WSL 2 com Ubuntu automaticamente (no Windows 11).

#### 1.2 Se necessário, instale manualmente:
- Habilite a funcionalidade WSL e Virtual Machine:
```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
- Reinicie o computador
- Instale a atualização do kernel do WSL 2:  
https://aka.ms/wsl2kernel
- Defina o WSL 2 como padrão:
```powershell
wsl --set-default-version 2
```
- Instale o Ubuntu pela Microsoft Store

Mais detalhes sempre atualizados em: https://learn.microsoft.com/pt-br/windows/wsl/install

Depois, abra o Ubuntu no menu iniciar e siga os próximos passos normalmente.

### 🍎 Caso você esteja usando macOS:

Você pode seguir com o terminal nativo do macOS. Recomendado usar o Homebrew para instalar as ferramentas:

#### 1.1 Instale o Homebrew (se ainda não tiver):
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 1.2 Instale ferramentas básicas:
```bash
brew install git terraform ansible node docker awscli azure-cli
```

> Obs: no macOS o Docker precisa do Docker Desktop. Baixe em: https://www.docker.com/products/docker-desktop

Verifique as versões:
```bash
git --version
terraform -version
ansible --version
docker --version
aws --version
az version
```

---

## 2. Git instalado e configurado

Instale o Git:
```bash
sudo apt update
sudo apt install git -y
```

Configure nome e email:
```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
```

Verifique:
```bash
git --version
```

---

## 3. Conta no GitHub criada

- Crie uma conta em: https://github.com
- Crie um repositório pessoal para os labs
- Configure a chave SSH (opcional): https://docs.github.com/pt/authentication/connecting-to-github-with-ssh

---

## 4. Docker instalado

Siga as instruções oficiais para instalar:
https://docs.docker.com/engine/install/ubuntu/

Verifique:
```bash
docker --version
```

Teste se o Docker está funcionando:
```bash
sudo docker run hello-world
```

---

## 5. AWS CLI, Azure CLI e Bicep CLI instaladas

### AWS CLI:
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### Azure CLI:
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
az version
```

Documentação oficial: https://learn.microsoft.com/pt-br/cli/azure/install-azure-cli

### Bicep CLI:
```bash
az bicep install
bicep --version
```

Documentação oficial: https://learn.microsoft.com/pt-br/azure/azure-resource-manager/bicep/install

Autentique-se:
```bash
aws configure
az login
```

---

## 6. Terraform instalado

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install terraform -y
terraform -version
```

---

## 7. Ansible instalado

```bash
sudo apt update
sudo apt install ansible -y
ansible --version
```

---

## 8. Node.js (para alguns labs com app containerizada)

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

---

## 9. Kubernetes local com Minikube (ou kind)

### Minikube:
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### kind (alternativa):
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

---

## 10. Visual Studio Code com extensões

- Instale o VS Code: https://code.visualstudio.com
- Instale as extensões:
  - Docker
  - Remote - WSL (se estiver usando WSL)
  - Terraform
  - YAML
  - GitHub Pull Requests

---

## Pronto!

Com esses itens instalados e configurados, você está preparado para iniciar todos os módulos do curso DevOps Automation.

