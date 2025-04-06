# 🧪 Linux Essencial para Automação

Este lab é focado nos comandos Linux mais usados em scripts de automação com Shell Scripts e pipelines.

Vamos praticar:

- Navegação
- Permissões
- Processos
- Pacotes
- Rede
- Redirecionamento `>`, `>>`, `|`
- Filtros com `jq`, `awk`, `grep`, `cut`, `sed`
- Uso de variáveis em shell scripts

---

## 🔧 Preparando ambiente de labs

Execute este bloco uma única vez para criar a estrutura base dos labs:

```bash
mkdir -p ~/labs/linux/projeto/{scripts,logs,dados}
cd ~/labs/linux/projeto
```

---

## Lab 1 - Navegação no terminal

```bash
pwd               # Mostra o caminho atual
ls -la            # Lista tudo com detalhes
cd scripts        # Entra na pasta de scripts
cd ..             # Volta para projeto
```

Ver estrutura:
```bash
tree ~/labs/linux/projeto
```

---

## Lab 2 - Permissões de arquivos e execução

```bash
cd ~/labs/linux/projeto/scripts
touch run.sh
chmod +x run.sh
ls -l
```

Adicione conteúdo no script:
```bash
echo -e '#!/bin/bash\necho "Script executado"' > run.sh
./run.sh
```

---

## Lab 3 - Processos e monitoração

```bash
top            # Pressione q para sair
ps aux | grep bash
```

Crie um processo em segundo plano:
```bash
sleep 300 &
ps aux | grep sleep
kill -9 <PID>   # Substitua <PID> pelo ID real
```

---

## Lab 4 - Instalação de pacotes

```bash
sudo apt update
sudo apt install -y curl git jq net-tools tree
```

Use os comandos instalados:
```bash
curl ifconfig.me
ip a
tree ~/labs/linux
```

---

## Lab 5 - Redirecionamento e arquivos

```bash
cd ~/labs/linux/projeto/dados
echo "linha 1" > resultado.txt
echo "linha 2" >> resultado.txt
cat resultado.txt
```

Filtrar dados:
```bash
cat resultado.txt | grep linha
```

---

## Lab 6 - Filtros de texto com `cut`, `awk`, `sed`

```bash
echo "nome,email,idade" | cut -d',' -f2

echo "usuario:x:1000:1000::/home/usuario:/bin/bash" | awk -F: '{print $1}'

echo "nome antigo" | sed 's/antigo/novo/'
```

---

## Lab 7 - Parse de JSON com `jq`

```bash
curl -s https://api.github.com/repos/docker/docker-ce | jq '.name, .language'
```

Filtrar um objeto:
```bash
curl -s https://api.github.com/repos/hashicorp/terraform | jq '{nome: .name, linguagem: .language}'
```

---

## Lab 8 - Variáveis em shell scripts

Crie um script:
```bash
cd ~/labs/linux/projeto/scripts
echo -e '#!/bin/bash\nNOME="DevOps"\necho "Bem-vindo, $NOME"' > saudacao.sh
chmod +x saudacao.sh
./saudacao.sh
```

Crie um script com entrada externa:
```bash
echo -e '#!/bin/bash\nARQUIVO=$1\necho "Rodando script..." > $ARQUIVO' > gravar.sh
chmod +x gravar.sh
./gravar.sh ../dados/resultado.txt
cat ../dados/resultado.txt
```

Execute este bloco uma única vez para criar a estrutura base dos labs:

```bash
mkdir -p ~/labs/linux/projeto/{scripts,logs,dados}
cd ~/labs/linux/projeto
```

---

## Lab 9 - Variáveis em shell scripts

### saudacao.sh
```bash
#!/bin/bash
NOME="DevOps"
echo "Bem-vindo, $NOME"
```

### gravar.sh
```bash
#!/bin/bash
ARQUIVO=$1
echo "Rodando script..." > $ARQUIVO
```

Execute:
```bash
cd ~/labs/linux/projeto/scripts
chmod +x saudacao.sh gravar.sh
./saudacao.sh
./gravar.sh ../dados/resultado.txt
cat ../dados/resultado.txt
```

---

## Lab 10 - Condições com if

### condicional.sh
```bash
#!/bin/bash
if [ "$1" == "ok" ]; then
  echo "Tudo certo"
else
  echo "Algo errado"
fi
```

Execute:
```bash
chmod +x condicional.sh
./condicional.sh ok
./condicional.sh erro
```

---

## Lab 11 - Estrutura de repetição com for

### loop.sh
```bash
#!/bin/bash
for i in 1 2 3; do
  echo "Executando $i"
done
```

Execute:
```bash
chmod +x loop.sh
./loop.sh
```

---

## Lab 12 - Leitura de arquivo linha a linha

Crie o arquivo:
```bash
echo -e "um\ndois\ntres" > ~/labs/linux/projeto/dados/linhas.txt
```

### leitor.sh
```bash
#!/bin/bash
while read linha; do
  echo "Linha: $linha"
done < ../dados/linhas.txt
```

Execute:
```bash
chmod +x leitor.sh
./leitor.sh
```

---

## Lab 13 - Validando se arquivo existe

### checar_arquivo.sh
```bash
#!/bin/bash
ARQ="../dados/resultado.txt"
if [ -f "$ARQ" ]; then
  echo "Arquivo existe: $ARQ"
else
  echo "Arquivo nao encontrado"
fi
```

Execute:
```bash
chmod +x checar_arquivo.sh
./checar_arquivo.sh
```

---

## Lab 14 - Exportar variável para outros scripts

### exporta.sh
```bash
#!/bin/bash
export NOME_GLOBAL="LinuxPro"
./usa_variavel.sh
```

### usa_variavel.sh
```bash
#!/bin/bash
echo "Usando variavel: $NOME_GLOBAL"
```

Execute:
```bash
chmod +x exporta.sh usa_variavel.sh
./exporta.sh
```
---