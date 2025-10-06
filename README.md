# 🚀 AWS - Automação com Lambda Function e S3

## 📌 Descrição do Projeto
Este projeto foi desenvolvido como parte do desafio da **DIO**, com o objetivo de **consolidar conhecimentos em automação de tarefas utilizando AWS Lambda e Amazon S3**, explorando a criação, integração e documentação de recursos em nuvem.  

O projeto demonstra como eventos do **Amazon S3** podem acionar **funções Lambda** para processar arquivos automaticamente, registrando resultados e possibilitando integrações com outros serviços como **DynamoDB**.  

Além disso, o projeto inclui práticas com **CloudFormation**, permitindo a criação de infraestrutura de forma automatizada, escalável e reprodutível.  

---

## 🎯 Objetivos de Aprendizagem
Ao concluir este desafio, você será capaz de:  

- Aplicar conceitos de **serverless computing** utilizando AWS Lambda.  
- Configurar **gatilhos de eventos no Amazon S3**.  
- **Documentar processos técnicos** de forma clara e estruturada.  
- Utilizar o **GitHub** como ferramenta para compartilhamento de conhecimento técnico.  
- Entender como o **CloudFormation** auxilia na automação de recursos.  
- Simular ambientes AWS localmente com **LocalStack**.  

---

## 🛠️ Arquitetura da Solução

**Fluxo resumido**:  
1. O usuário faz **upload de um arquivo** no bucket S3.  
2. O evento de upload aciona uma **função Lambda**.  
3. A função processa o arquivo (ex.: gera log, extrai dados, redimensiona imagem, etc.).  
4. Opcionalmente, os resultados são armazenados no **DynamoDB**.  

Usuário → S3 (Upload) → Lambda (Processamento) → DynamoDB (Metadados)


---

## ⚡ Serviços Utilizados
- **Amazon S3** → Armazenamento de objetos.  
- **AWS Lambda** → Funções serverless acionadas por eventos.  
- **Amazon DynamoDB** → Armazenamento de metadados (opcional).  
- **IAM** → Permissões e papéis para execução.  
- **AWS CloudFormation** → Automação de criação de recursos.  
- **LocalStack** → Ambiente local para testes de serviços AWS.  

---

---

## 📝 Exemplo de Lambda Function (Python)

```python
import json

def lambda_handler(event, context):
    # Captura o nome do bucket e do arquivo enviado
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    print(f"Arquivo recebido: {key} no bucket: {bucket}")

    # Exemplo: resposta simples de sucesso
    return {
        'statusCode': 200,
        'body': json.dumps(f"Arquivo {key} processado com sucesso no bucket {bucket}!")
    }

````

🏗️ Exemplo de Template CloudFormation (YAML)
````
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: lambda-s3-dio-desafio

  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: LambdaExecutionRoleDIO
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: ProcessaUploadS3
      Runtime: python3.9
      Role: !GetAtt LambdaExecutionRole.Arn
      Handler: lambda_function.lambda_handler
      Code:
        ZipFile: |
          import json
          def lambda_handler(event, context):
              bucket = event['Records'][0]['s3']['bucket']['name']
              key = event['Records'][0]['s3']['object']['key']
              print(f"Arquivo {key} processado do bucket {bucket}")
              return {"statusCode": 200, "body": "Processado com sucesso"}

  S3BucketNotification:
    Type: AWS::S3::BucketNotification
    Properties:
      Bucket: !Ref S3Bucket
      NotificationConfiguration:
        LambdaConfigurations:
          - Event: "s3:ObjectCreated:*"
            Function: !GetAtt LambdaFunction.Arn

````
🖥️ Exemplo com LocalStack

Criar bucket no LocalStack:
````
awslocal s3 mb s3://meu-bucket-local

````
Criar função Lambda:
````
awslocal lambda create-function \
  --function-name HelloWorld \
  --runtime python3.9 \
  --role arn:aws:iam::000000000000:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

````
📘 Anotações e Insights


O S3 é ideal para armazenamento de objetos, com alta durabilidade e escalabilidade.

O Lambda elimina a necessidade de servidores, cobrando apenas pela execução.

A combinação S3 + Lambda + DynamoDB cria pipelines de dados altamente automatizados.

O CloudFormation facilita a padronização e automação da infraestrutura.

O LocalStack é excelente para reduzir custos e testar localmente.


✅ Conclusão

Este desafio consolidou os principais conceitos de automação em nuvem com AWS, 
explorando como eventos podem disparar funções serverless para processar dados de forma rápida, 
escalável e sem gerenciamento de servidores.

O projeto serve como base para implementações reais, podendo ser expandido
com integrações a APIs, workflows de dados e monitoramento com CloudWatch.
