# 🐳 Lab Docker: Do Básico ao Avançado

Este conjunto de labs práticos foi criado para te ensinar a usar Docker de forma eficiente no dia a dia de desenvolvimento e automação. Começaremos pelos comandos básicos e evoluiremos até troubleshooting, cópia de arquivos, criação de imagens personalizadas e uso do `docker-compose`.

---

## Pré-requisitos

- Docker instalado ([Instruções](https://docs.docker.com/get-docker/))
- Terminal Linux ou WSL/Mac

Verifique a versão instalada:
```bash
docker --version
```

Verifique se o serviço está ativo:
```bash
sudo systemctl status docker
```

---

## Lab Docker – Versão para SSH (Ubuntu)

### 🎯 Objetivo

Executar os comandos básicos do Docker sem perder acesso ao terminal via SSH

---

## Lab 2 – Comandos Básicos com Segurança Remota

### 1. Baixar imagem do Nginx

```bash
docker pull nginx
```

### 2. Rodar container em modo detached

```bash
docker run -d --name webserver -p 8080:80 nginx
```

### 3. Verificar containers rodando

```bash
docker ps
```

### 4. Verificar se está funcionando

```bash
curl http://localhost:8080
```

### 5. Acessar o terminal do container

```bash
docker exec -it webserver bash
```

### 6. Parar e remover o container

```bash
docker stop webserver
docker rm webserver
```

### 7. Remover a imagem (opcional)

```bash
docker rmi nginx
```

### Evite comandos interativos diretos em SSH

```bash
docker run -it ubuntu bash
```

📌 Sempre prefira `-d` e `docker exec` para ambientes remotos


---

## Lab 3 – Reutilizando containers com `exec`

### 1. Ver containers existentes
```bash
docker ps -a
```

### 2. Iniciar um container parado
```bash
docker start <id ou nome>
```

### 3. Acessar o terminal de um container já iniciado
```bash
docker exec -it <id ou nome> bash
```

---

## Lab 4 – Limpeza e manutenção

### 1. Remover container específico
```bash
docker rm <id ou nome>
```

### 2. Remover imagem específica
```bash
docker rmi <imagem>
```

### 3. Limpar containers, volumes e imagens não utilizados
```bash
docker system prune -a
```

---

## Lab 5 – Copiar arquivos entre host e container

### 1. Copiar do host para o container
```bash
docker cp meu_script.sh <id>:/root/
```

### 2. Copiar do container para o host
```bash
docker cp <id>:/root/arquivo.log ./arquivo.log
```

---

## Lab 6 – Visualização de logs e debug

## 📁 1. Criar estrutura do projeto

```bash
mkdir -p ~/labs/docker/app
cd ~/labs/docker/app
```

* `mkdir -p` cria o diretório principal do projeto.
* `cd` navega até o diretório principal.

---

## ✍️ 2. Criar arquivos da aplicação

### Criar o diretório e o arquivo principal:

```bash
mkdir -p src
touch src/index.js
```

* `mkdir -p src` garante que o diretório `src` existe.
* `touch src/index.js` cria o arquivo principal da aplicação.

### Editar o conteúdo do `index.js`:

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from a professional Docker container!');
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

* Esse código cria um servidor web simples em Node.js que responde com texto puro.

---

## 📦 3. Criar o arquivo `package.json`

### Criar o arquivo:

```bash
touch package.json
```

### Adicionar o conteúdo abaixo ao `package.json`:

```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {}
}
```

* Esse arquivo define as informações básicas da aplicação e o comando para iniciar.

---
```
Pronto! Agora você tem uma aplicação Node.js com estrutura organizada, pronta para ser dockerizada. Na próxima etapa, vamos criar o `Dockerfile` e subir esse app em um container.

Deseja seguir para o Dockerfile?

```


### 4. Criar Dockerfile otimizado
```Dockerfile
# Etapa 1: build da aplicação (instalação de dependências)
FROM node:18-alpine AS builder
WORKDIR /app
COPY src/package.json ./
RUN npm install
COPY src/ .

# Etapa 2: imagem final
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
EXPOSE 3000
CMD ["npm", "start"]
```

### 5. Build da imagem
```bash
docker build -t devopsautomation:latest .
```

### 6. Executar a imagem
```bash
docker run -p 3000:3000 devopsautomation:latest
```

Você pode acessar a aplicação no navegador: [http://localhost:3000](http://localhost:3000)

---

## Lab 8 – Criando imagem multistage

### 1. Criar estrutura
```bash
mkdir -p ~/labs/docker/multistage
cd ~/labs/docker/multistage
```

### 2. Criar arquivo do app
```bash
echo 'console.log("Build otimizado!")' > index.js
```

### 3. Criar Dockerfile multistage:
```Dockerfile
FROM node:18-alpine AS build
WORKDIR /src
COPY index.js .

FROM node:18-alpine
WORKDIR /app
COPY --from=build /src/index.js .
CMD ["node", "index.js"]
```

### 4. Build e execução:
```bash
docker build -t devopsautomation:multi .
docker run devopsautomation:multi
```

---

## Lab 9 – Enviar imagem para o Docker Hub

### 1. Autenticar no Docker Hub
```bash
docker login
```

### 2. Taguear a imagem antes de enviar
```bash
docker tag devopsautomation:multi seuusuario/devopsautomation:1.0
```

### 3. Enviar a imagem
```bash
docker push seuusuario/devopsautomation:1.0
```

---

## Lab 10 – Docker Compose com rede e múltiplos containers

### 1. Criar diretório do lab
```bash
mkdir -p ~/labs/docker/network-demo
cd ~/labs/docker/network-demo
```

### 2. Criar `docker-compose.yml`
Crie o arquivo:
```bash
touch docker-compose.yml
```
Abra e cole o conteúdo a seguir:

```yaml
version: '3.8'

services:
  app1:
    image: nginx
    container_name: app1
    networks:
      - appnet

  app2:
    image: nginx
    container_name: app2
    networks:
      - appnet

  app3:
    image: nginx
    container_name: app3
    networks:
      - appnet

networks:
  appnet:
    driver: bridge
```

### 3. Subir os containers
```bash
docker compose up -d
```

### 4. Verificar os containers e a rede
```bash
docker compose ps
docker network inspect network-demo_appnet
```

Você verá os IPs dos três containers atribuídos automaticamente pela rede bridge `appnet`.

### 5. Acessar os containers e testar conectividade

Entre no `app1`:
```bash
docker exec -it app1 bash
```

Dentro do container:
```bash
apt update && apt install -y iputils-ping
ping app2 -c 3
ping app3 -c 3
exit
```

### 6. Parar e remover os serviços
```bash
docker compose down
```

### Fim do Lab 11

---

## Lab 11 – Testando persistência de dados com volumes

### 1. Criar diretório do lab
```bash
mkdir -p ~/labs/docker/volumes-demo
cd ~/labs/docker/volumes-demo
```

### 2. Criar `docker-compose.yml`
```bash
touch docker-compose.yml
```

Abra o arquivo e cole o conteúdo:

```yaml
version: '3.8'

services:
  db:
    image: mysql:5.7
    container_name: mysqldb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb
    volumes:
      - dbdata:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  dbdata:
```

### 3. Subir o banco com volume
```bash
docker compose up -d
```

### 4. Acessar o banco
```bash
docker exec -it mysqldb bash
mysql -uroot -proot
```

### 5. Criar uma tabela para teste
```sql
USE testdb;
CREATE TABLE users (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100));
INSERT INTO users (name) VALUES ('Maria'), ('João');
SELECT * FROM users;
```

### 6. Remover os containers
```bash
docker compose down
```


---