# ToggleMaster

ToggleMaster é uma arquitetura de microsserviços para gerenciamento de **feature flags** com rollout gradual, segmentação de usuários e análise de uso. Ideal para aplicações que precisam liberar funcionalidades de forma controlada e segura.

---

## Arquitetura

O projeto é composto por 5 microsserviços:

| Serviço             | Linguagem | Função                                                                 |
|---------------------|-----------|------------------------------------------------------------------------|
| `auth-service`      | Go        | Criação e validação de chaves de API                                  |
| `flag-service`      | Python    | CRUD de feature flags                                                 |
| `targeting-service` | Python    | Regras de segmentação (ex: porcentagem de usuários, localização)      |
| `evaluation-service`| Go        | Avaliação em tempo real de flags para usuários, com cache e envio SQS |
| `analytics-service` | Python    | Worker que consome eventos do SQS e grava no DynamoDB                 |

---
## Fluxo de avaliação
```sh
O cliente chama evaluation-service com user_id e flag_name.
O serviço consulta o cache Redis.
Se necessário, busca dados no flag-service e targeting-service.
Avalia se a flag está ativa para o usuário.
Retorna true ou false.
Envia evento para o SQS.
analytics-service consome e grava no DynamoDB.
```
## Rodando localmente

### Pré-requisitos
- Docker + Docker Compose

### Passos
```bash
\ Clone este repo
\ extraia o conteúdo do arquivo ToggleMaster.zip
```
### IMPORTANTE:
É necessário configurar as credenciais AWS no .env
para isso, basta rodar o comando abaixo e pegar as credenciais

```sh
cat .aws/credentials 
```

(necessário criar a fila SQS e DynamoDB no usuário da AWS Academy)

### Criar SQS:
```sh
aws sqs create-queue --queue-name togglemaster-events
```

### Criar DynamoDB:
```sh
aws dynamodb create-table \
    --table-name ToggleMasterAnalytics \
    --attribute-definitions AttributeName=event_id,AttributeType=S \
    --key-schema AttributeName=event_id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
```
Basta rodar os comandos no AWS CLI do academy

### Suba o serviço 
Depois do .env configurado basta subir o docker compose
```bash
docker compose up -d --build
```


