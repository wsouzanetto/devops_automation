# 📘 Introdução: DevOps e o uso do vi para automação

DevOps é uma abordagem que une o desenvolvimento de software (Dev) com as operações de infraestrutura (Ops), promovendo automação, integração contínua e entrega contínua (CI/CD). Com ele, conseguimos entregar valor de forma mais rápida, confiável e segura. Dentro dessa cultura, ferramentas de linha de comando e editores como o `vi` são essenciais para realizar configurações rápidas, editar scripts diretamente em servidores e interagir com ambientes de produção ou pipelines.

Muitos profissionais DevOps utilizam o terminal no dia a dia. Seja acessando servidores via SSH, criando arquivos de configuração, ajustando pipelines, depurando scripts ou apenas verificando logs. Por isso, dominar comandos básicos e intermediários do editor `vi` não é apenas útil — é fundamental.

A seguir, vamos apresentar conceitos e comandos importantes relacionados ao uso do `vi` e sua aplicação no contexto DevOps, especialmente ao executar os labs propostos anteriormente.

## Por que usar o `vi`?
- Já vem instalado na maioria das distribuições Linux
- Funciona em qualquer terminal, mesmo em ambientes mínimos
- Permite edição rápida sem interface gráfica
- Ideal para editar arquivos de configuração, YAML, scripts bash e logs

## Exemplos de arquivos que DevOps edita com `vi`
- `.bashrc`, `.bash_profile`
- `Dockerfile`
- `docker-compose.yml`
- `main.tf` (Terraform)
- `playbook.yml` (Ansible)
- `*.sh` (scripts bash)
- `config.yaml`, `values.yaml`
- Logs: `/var/log/syslog`, `/var/log/nginx/error.log`

## Dicas para não sofrer com o `vi`

### 1. Entrar no modo correto antes de colar comandos
Quando colamos muitos comandos com indentação (como scripts bash ou YAML), o `vi` pode "quebrar" o alinhamento. Para evitar isso, usamos o comando:

```bash
:set paste
```

Isso deve ser executado **antes de entrar no modo de inserção** (`i`). Assim o vi entende que o conteúdo será colado e não tenta aplicar autoindentação.

Após colar, você pode voltar ao modo normal com:
```bash
:set nopaste
```

### 2. Atalhos comuns
- `i` – entrar no modo de inserção
- `ESC` – sair do modo de edição
- `:w` – salvar
- `:q` – sair
- `:wq` – salvar e sair
- `:q!` – sair sem salvar

### 3. Para copiar e colar
- `yy` – copia a linha atual
- `p` – cola abaixo da linha atual
- `dd` – apaga a linha
- `u` – desfaz última ação
- `Ctrl + r` – refaz (quando disponível)

### 4. Buscar e substituir (muito útil em DevOps)
Buscar por uma palavra:
```bash
/variavel
```
Substituir em todo o arquivo:
```bash
:%s/old/new/g
```

### 5. Adicionar linha abaixo rapidamente
Com o cursor em qualquer linha:
- Pressione `o` – cria nova linha abaixo e já entra em modo de inserção

### 6. Ir para o final ou início
- `G` – vai para a última linha
- `gg` – vai para a primeira linha

### 7. Editar múltiplos arquivos
```bash
vi arquivo1.txt arquivo2.sh
```
Dentro do `vi`, use `:n` para ir para o próximo arquivo

---

## 📗 Lab Prático com arquivos reais

### 1. Crie os arquivos com `touch`
```bash
cd ~/labs/linux/vi
mkdir -p ~/labs/linux/vi
cd ~/labs/linux/vi
touch script.sh docker-compose.yml config.yaml
```

### 2. Edite o `script.sh` com `vi`
```bash
vi script.sh
```

#### Cole o seguinte conteúdo (use `:set paste` antes de `i`):
```bash
#!/bin/bash

NOME="DevOps"
echo "Bem-vindo, $NOME"

for i in 1 2 3; do
  echo "Item: $i"
done
```

Salve com `:wq`

---

### 3. Edite o `docker-compose.yml`
```bash
vi docker-compose.yml
```

#### Conteúdo:
```yaml
version: '3.8'
services:
  app:
    image: nginx
    ports:
      - "8080:80"
```

Salve com `:wq`

---

### 4. Edite o `config.yaml`
```bash
vi config.yaml
```

#### Conteúdo:
```yaml
database:
  host: localhost
  port: 5432
  user: admin
  password: senha123
```

Salve com `:wq`

---

### 5. Comandos extras para praticar no `script.sh`
```bash
vi script.sh
```

- Substituir `echo` por `printf`:
```bash
:%s/echo/printf/g
```
- Ir para a última linha e adicionar:
```bash
G
o
echo "Script finalizado com sucesso"
ESC
:wq
```

---
