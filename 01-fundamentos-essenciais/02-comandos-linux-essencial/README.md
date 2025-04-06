# Linux Essencial para Automação

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

## Preparando ambiente de labs

Execute este bloco uma única vez para criar a estrutura base dos labs:

```bash
mkdir -p ~/labs/linux/projeto/{scripts,logs,dados}
cd ~/labs/linux/projeto
```

---

## Lab 8 - Variáveis em shell scripts

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

## Lab 9 - Condições com if

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

## Lab 10 - Estrutura de repetição com for

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

## Lab 11 - Leitura de arquivo linha a linha

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

## Lab 12 - Validando se arquivo existe

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

## Lab 13 - Exportar variável para outros scripts

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