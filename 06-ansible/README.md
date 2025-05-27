## Organização dos Labs Ansible – Estrutura Unificada

### Objetivo:

Consolidar todos os labs de Ansible em uma estrutura única dentro do diretório `~/ansible-lab/`, utilizando um único inventário e subpastas opcionais para organização dos playbooks.

---

### Estrutura de Diretório Sugerida:

```bash
~/ansible-lab/
├── hosts                    # Inventário único
├── playbooks/              # Playbooks organizados por lab
│   ├── setup-basico.yaml
│   └── outros futuros...
```

---

## Lab Ansible 01 – Criando as VMs no Azure

### Objetivo:

Criar duas VMs Ubuntu 20.04 no Azure para servir de controller e target para automação com Ansible

---

### Etapas:

#### 1. Criar Resource Group

```bash
az group create \
  --name ansible-lab-rg \
  --location eastus
```

---

#### 2. Criar a VM controller

```bash
az vm create \
  --name ansible-controller \
  --resource-group ansible-lab-rg \
  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest \
  --admin-username azureuser \
  --admin-password 'SenhaForte123!@#' \
  --authentication-type password \
  --public-ip-sku Basic
```

---

#### 3. Criar a VM target

```bash
az vm create \
  --name ansible-target \
  --resource-group ansible-lab-rg \
  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest \
  --admin-username azureuser \
  --admin-password 'SenhaForte123!@#' \
  --authentication-type password \
  --public-ip-sku Basic
```

---

#### 4. Abrir a porta 22 (SSH) nas duas VMs

```bash
az vm open-port --resource-group ansible-lab-rg --name ansible-controller --port 22
az vm open-port --resource-group ansible-lab-rg --name ansible-target --port 22
```

---

#### (Opcional) Expor porta de aplicação (ex: 8081)

```bash
az network nsg rule create \
  --resource-group ansible-lab-rg \
  --nsg-name ansible-targetNSG \
  --name Allow-8081 \
  --priority 1002 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 8081
```

---

### Resultado esperado:

* Duas VMs criadas no Azure (controller e target)
* Acesso SSH liberado nas duas máquinas
* Ambiente pronto para instalação e testes com Ansible

---

## Lab Ansible 02 – Instalar Ansible e testar conexão

### Objetivo:

Instalar o Ansible e sshpass na controller, configurar o inventário e testar a conexão com a target

---

### Etapas:

#### 1. Conectar na VM controller e organizar estrutura

```bash
az vm list-ip-addresses \
  --resource-group ansible-lab-rg \
  --name ansible-controller \
  --query "[].virtualMachine.network.publicIpAddresses[].ipAddress" \
  -o tsv

az vm list-ip-addresses \
  --resource-group ansible-lab-rg \
  --name ansible-target \
  --query "[].virtualMachine.network.publicIpAddresses[].ipAddress" \
  -o tsv

ssh azureuser@<IP_DA_CONTROLLER>
mkdir -p ~/ansible-lab/playbooks
cd ~/ansible-lab
```

---

#### 2. Instalar dependências

```bash
sudo apt update
sudo apt install -y ansible sshpass
```

---

#### 3. Exportar variável de ambiente temporária

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

> Essa variável desativa a checagem da chave SSH
> Validade: apenas na sessão atual

---

#### 4. Criar o arquivo de inventário

```bash
vi ~/ansible-lab/hosts
```

Conteúdo:

```ini
[targets]
vmtarget ansible_host=<IP_DA_TARGET> ansible_user=azureuser ansible_password="SenhaForte123!@#" ansible_python_interpreter=/usr/bin/python3
```

---

#### 5. Testar conexão com módulo ping

```bash
ansible -i hosts targets -m ping
```

---

### Resultado esperado:

* Conexão bem-sucedida com a target
* Resposta `pong` do Ansible

---

## Lab Ansible 03 – Criando e executando seu primeiro playbook

### Objetivo:

Criar um playbook para atualizar pacotes e instalar utilitários na máquina target

---

### Etapas:

#### 1. Criar arquivo do playbook

```bash
cd ~/ansible-lab/playbooks
vi setup-basico.yaml
```

Conteúdo:

```yaml
---
- name: Configurar VM com pacotes básicos
  hosts: targets
  become: true

  tasks:
    - name: Atualizar lista de pacotes
      apt:
        update_cache: yes

    - name: Instalar pacotes úteis
      apt:
        name:
          - htop
          - curl
          - git
        state: present
```

---

#### 2. Verificar variável de ambiente

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

---

#### 3. Executar o playbook

```bash
cd ~/ansible-lab
ansible-playbook -i hosts playbooks/setup-basico.yaml
```

---

#### 4. Validar resultados

```bash
ansible -i hosts targets -m shell -a "which htop"
ansible -i hosts targets -m shell -a "htop --version"
```

---

### Resultado esperado:

* Pacotes atualizados e instalados com sucesso
* Primeira execução: `changed: true`
* Segunda execução: `ok: true`

Ambiente funcional para automações com Ansible

## Lab Ansible 04 – Instalando o Docker e executando um container Java

### Objetivo:

Instalar o Docker e rodar a aplicação `iesodias/java-api:latest` na máquina target usando Ansible

---

### Etapas:

#### 1. Criar o arquivo do playbook

```bash
cd ~/ansible-lab/playbooks
nano docker-api.yaml
```

Conteúdo:

```yaml
---
- name: Instalar Docker e rodar aplicação Java
  hosts: targets
  become: true

  tasks:
    - name: Atualizar pacotes
      apt:
        update_cache: yes

    - name: Instalar dependências do Docker
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present

    - name: Adicionar chave GPG do Docker
      shell: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

    - name: Adicionar repositório do Docker
      shell: add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

    - name: Atualizar pacotes novamente
      apt:
        update_cache: yes

    - name: Instalar Docker
      apt:
        name: docker-ce
        state: present

    - name: Habilitar e iniciar o Docker
      systemd:
        name: docker
        enabled: yes
        state: started

    - name: Instalar pip para Python 3
      apt:
        name: python3-pip
        state: present

    - name: Instalar SDK do Docker para Python
      pip:
        name: docker

    - name: Adicionar usuário ao grupo docker
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Rodar o container com a aplicação Java
      docker_container:
        name: java-api
        image: iesodias/java-api:latest
        state: started
        restart_policy: always
        ports:
          - "8081:8081"
```

---

#### 2. Verificar variável de ambiente

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

---

#### 3. Executar o playbook

```bash
cd ~/ansible-lab
ansible-playbook -i hosts playbooks/docker-api.yaml
```

---

#### 4. Validar resultado

```bash
ansible -i hosts targets -m shell -a "docker ps"
```

Você deve ver a imagem `iesodias/java-api:latest` em execução

---

### Resultado esperado:

* Docker instalado corretamente na VM target
* Container `java-api` em execução na porta 8081
* Aplicação acessível via IP público da VM target


## Lab Ansible 06 – Gerenciar usuário, permissões e estrutura de pastas

### Objetivo:

Criar um novo usuário (ex: devops), conceder permissões administrativas e configurar diretórios específicos com as permissões corretas

---

### Etapas:

#### 1. Criar o arquivo do playbook

```bash
cd ~/ansible-lab/playbooks
nano usuario-devops.yaml
```

#### Conteúdo do playbook:

```yaml
---
- name: Criar usuário e gerenciar permissões
  hosts: targets
  become: true

  tasks:
    - name: Criar usuário devops com senha e bash
      user:
        name: devops
        password: "{{ 'Devops123!@#' | password_hash('sha512') }}"
        shell: /bin/bash
        groups: sudo,docker
        append: yes
        state: present

    - name: Criar diretório de aplicação
      file:
        path: /opt/app
        state: directory
        owner: devops
        group: devops
        mode: '0755'

    - name: Criar diretório de logs
      file:
        path: /opt/logs
        state: directory
        owner: devops
        group: devops
        mode: '0755'

    - name: Criar arquivo de teste no diretório de app
      copy:
        content: "Aplicação iniciada em {{ ansible_date_time.date }} {{ ansible_date_time.time }}\n"
        dest: /opt/app/status.txt
        owner: devops
        group: devops
        mode: '0644'

    - name: Criar arquivo de log simulado
      shell: echo "log iniciado em $(date)" > /opt/logs/inicial.log
```

---

### Executar o playbook:

```bash
cd ~/ansible-lab
ansible-playbook -i hosts playbooks/usuario-devops.yaml
```

---

### Validar resultado:

```bash
ansible -i hosts targets -m shell -a "ls -l /opt"
ansible -i hosts targets -m shell -a "cat /opt/app/status.txt"
ansible -i hosts targets -m shell -a "id devops"
```

---

### Resultado esperado:

* Usuário `devops` criado com senha e shell bash
* Permissões de `sudo` e acesso ao `docker`
* Estrutura `/opt/app` e `/opt/logs` criada com dono correto
* Arquivos `status.txt` e `inicial.log` gerados e preenchidos



## 💣 Lab Final – Destruir o ambiente Ansible no Azure

### 🌟 Objetivo:

Remover todas as VMs, IPs, discos e recursos criados no laboratório do Ansible no Azure

---

### 🛠️ Etapas:

#### 🔎 1. Verifique os recursos criados (opcional)

```bash
az resource list --resource-group ansible-lab-rg --output table
```

> Isso mostra todos os recursos dentro do grupo `ansible-lab-rg`, incluindo VMs, IPs, discos e mais

---

#### 🗑️ 2. Apagar o Resource Group (e tudo dentro dele)

```bash
az group delete --name ansible-lab-rg --yes --no-wait
```

> Explicando:
>
> * `--yes`: confirma automaticamente a exclusão
> * `--no-wait`: roda o processo em segundo plano, sem travar o terminal

---

### ✅ Resultado esperado:

* As VMs `ansible-controller` e `ansible-target` são removidas
* IPs públicos, discos, NICs, NSGs e VNet também são apagados
* Nenhum recurso do lab permanece no Azure