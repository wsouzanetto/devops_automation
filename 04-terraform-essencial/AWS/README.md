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

Quer que eu detalhe algum desses tópicos avançados?

