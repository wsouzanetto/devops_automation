# Lab AWS - Criando Bucket S3 com Configurações de Segurança

## Objetivo  
Criar um bucket S3 com:  
- Versionamento ativado  
- Política de bloqueio público  
- Logging de acesso  

---

## Pré-requisitos  
- Conta AWS com permissões para S3  
- [AWS CLI instalada](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)  
- Opcional: Terraform instalado (versão ao final)  

---

## Passo a Passo (Console AWS)  

### 1. Acessar o Console S3  
1. Faça login no [Console AWS](https://console.aws.amazon.com)  
2. Busque por **"S3"** e clique no serviço  

### 2. Criar Bucket  
1. Clique em **"Criar bucket"**  
2. Preencha:  
   ```plaintext
   Nome do bucket: devops-labs-bucket-<seu-nome> (deve ser único globalmente)  
   Região: us-east-1 (N. Virginia) ou escolha sua região  
   ```

### 3. Configurar Opções  
```plaintext
✅ Ativar versionamento  
✅ Bloquear todo o acesso público (recomendado)  
🔒 Criptografia: AES-256 (SSE-S3) - Padrão  
```

### 4. Política de Acesso (Exemplo)  
Na aba "Permissões", cole esta política:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/seu-usuario"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::devops-labs-bucket-<seu-nome>/*"
    }
  ]
}
```

### 5. Ativar Logging (Opcional)  
Na aba "Propriedades" > "Logging de acesso ao servidor"  
Especifique outro bucket para logs (ou crie um novo)  

### 6. Revisar e Criar  
Clique em "Criar bucket"  

---

## Validação via AWS CLI

### 1. Listar buckets
```bash
aws s3 ls
```

### 2. Upload de arquivo teste
```bash
echo "Teste DevOps Lab" > teste.txt
aws s3 cp teste.txt s3://devops-labs-bucket-<seu-nome>/
```

### 3. Verificar versionamento
```bash
aws s3api list-object-versions --bucket devops-labs-bucket-<seu-nome>
```

---

## Versão Terraform (opcional)

### 1. Criar `s3.tf`
```hcl
resource "aws_s3_bucket" "devops_lab" {
  bucket = "devops-labs-bucket-<seu-nome>"
  acl    = "private"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }

  tags = {
    Lab = "DevOps-S3"
  }
}
```

### 2. Aplicar
```bash
terraform init && terraform apply
```

---

## Boas Práticas

🔐 Sempre use políticas IAM granulares (evite "Action": "s3:*")  
💸 Ative Lifecycle Rules para arquivos temporários  
🛡️ Use Bucket Policies + IAM em conjunto para segurança  

---

## Dica Extra:  
Para um bucket **production**, ative:  
- **Object Lock** (compliance)  
- **Cross-Region Replication**  
- **CloudTrail logging**  

---

# Lab AWS S3 com Terraform - Estrutura Profissional

## Estrutura de Arquivos
```plaintext
s3-lab/
├── main.tf          # Recursos principais
├── variables.tf     # Variáveis de entrada
├── outputs.tf       # Saídas do módulo
├── versions.tf      # Versões de providers
└── terraform.tfvars # Valores das variáveis (opcional)
```

### 1. versions.tf
```hcl
terraform {
  required_version = ">= 1.3.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 2. variables.tf
```hcl
variable "bucket_name" {
  description = "Nome único global para o bucket S3"
  type        = string
}

variable "environment" {
  description = "Ambiente (dev/stg/prod)"
  type        = string
  default     = "dev"
}

variable "enable_versioning" {
  description = "Ativar versionamento de objetos"
  type        = bool
  default     = true
}

variable "allowed_ips" {
  description = "Lista de IPs com acesso ao bucket"
  type        = list(string)
  default     = []
}
```

### 3. terraform.tfvars
```hcl
bucket_name       = "devops-labs-bucket-12345" # Substitua pelo seu nome único
environment       = "dev"
enable_versioning = true
allowed_ips       = ["189.34.123.45/32"] # Substitua pelo seu IP
```

### 4. main.tf
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    Terraform   = "true"
  }
}

resource "aws_s3_bucket_versioning" "this" {
  count = var.enable_versioning ? 1 : 0

  bucket = aws_s3_bucket.this.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_public_access_block" "this" {
  bucket = aws_s3_bucket.this.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_policy" "this" {
  count = length(var.allowed_ips) > 0 ? 1 : 0

  bucket = aws_s3_bucket.this.id
  policy = data.aws_iam_policy_document.bucket_policy[0].json
}

data "aws_iam_policy_document" "bucket_policy" {
  count = length(var.allowed_ips) > 0 ? 1 : 0

  statement {
    principals {
      type        = "AWS"
      identifiers = ["*"]
    }

    effect = "Deny"

    actions = [
      "s3:*",
    ]

    resources = [
      aws_s3_bucket.this.arn,
      "${aws_s3_bucket.this.arn}/*"
    ]

    condition {
      test     = "NotIpAddress"
      variable = "aws:SourceIp"
      values   = var.allowed_ips
    }
  }
}
```

### 5. outputs.tf
```hcl
output "bucket_arn" {
  description = "ARN do bucket criado"
  value       = aws_s3_bucket.this.arn
}

output "bucket_domain_name" {
  description = "Domínio do bucket"
  value       = aws_s3_bucket.this.bucket_domain_name
}

output "bucket_regional_domain_name" {
  description = "Domínio regional do bucket"
  value       = aws_s3_bucket.this.bucket_regional_domain_name
}
```

---

## Como Executar
```bash
# Inicializar
terraform init

# Verificar plano
terraform plan

# Aplicar configuração
terraform apply

# Destruir recursos (quando necessário)
terraform destroy
```

---

## Melhorias Implementadas
- Separação clara de responsabilidades por arquivo
- Variáveis parametrizáveis com valores padrão
- Política de segurança baseada em condições (IP)
- Outputs úteis para integração com outros sistemas
- Versionamento explícito de providers

---

### Dica de Uso:
1. Para ambientes diferentes (dev/stg/prod), crie arquivos `.tfvars` separados:
   - `dev.tfvars`
   - `production.tfvars`
2. Aplique com:
```bash
terraform apply -var-file="production.tfvars"
```

Quer que eu adicione algum recurso específico como:



