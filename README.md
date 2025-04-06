# DevOps Automation - Grade do Curso Prático

Abaixo está a estrutura organizada por módulos, com foco direto na prática e aplicação real.

---

## 🔧 Módulo 1 – Fundamentos Essenciais
**Objetivo:** nivelar o conhecimento da turma e garantir o domínio das ferramentas básicas

- Introdução ao curso e ao mundo DevOps
- Git na prática:
  - clone
  - add
  - commit
  - push
  - branch
  - merge
  - pull request no GitHub
- Comandos Linux essenciais:
  - navegação
  - permissões
  - processos
  - pacotes
  - rede

---

## 🐳 Módulo 2 – Containers com Docker
**Objetivo:** preparar o ambiente com containers e entender como usar imagens

- Conceitos de container na prática
- Imagens vs containers
- Docker CLI (build, run, exec, ps, stop, logs, etc)
- Dockerfile simples e docker-compose básico
- **Lab:** containerizando uma app simples

---

## ☁️ Módulo 3 – Primeiros Passos na Nuvem (AWS e Azure)
**Objetivo:** aprender a subir recursos básicos na mão antes de automatizar

- Overview dos principais serviços da AWS e Azure
- Criar máquina virtual e storage via portal e CLI (ambas clouds)
- **Lab:** subir um script de provisionamento simples (bash ou PowerShell)

---

## 🔁 Módulo 4 – Automação com Terraform
**Objetivo:** provisionar infraestrutura como código nas nuvens

- Conceito de IaC
- Estrutura de um projeto Terraform
- Providers AWS e Azure
- Variáveis e outputs
- **Lab:** provisionar uma VM com Terraform em cada nuvem

---

## 🧰 Módulo 5 – Configuração com Ansible
**Objetivo:** configurar servidores automaticamente

- Conceito de configuração como código
- Inventário e playbooks
- Módulos principais
- **Lab:** instalar e configurar uma aplicação em VM provisionada no Terraform

---

## 🤖 Módulo 6 – CI/CD com GitHub Actions
**Objetivo:** rodar pipelines simples com GitHub Actions

- Conceito de pipeline
- Actions e workflows
- **Lab:** build e deploy de app em container com GitHub Actions

---

## 📦 Módulo 7 – CI/CD com Azure DevOps
**Objetivo:** mostrar o poder do Azure DevOps para projetos maiores

- Pipelines YAML
- Variáveis e stages
- **Lab:** criar uma esteira de deploy em Azure com Terraform + Ansible

---

## ☸️ Módulo 8 – Fundamentos de Kubernetes
**Objetivo:** entender e testar os conceitos essenciais

- O que é Kubernetes
- Pods, Services, Deployments
- **Lab:** subir app containerizada localmente (minikube ou kind)