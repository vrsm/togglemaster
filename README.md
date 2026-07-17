# 🚀 ToggleMaster - Tech Challenge Fase 2

Implementação de uma plataforma de **Feature Toggle** baseada em microsserviços utilizando **Docker Compose**, **Kubernetes (Amazon EKS)**, **Amazon SQS**, **Amazon DynamoDB**, **Redis** e **PostgreSQL**.

---

## 🎥 Demonstração

**YouTube**

https://youtu.be/hQbXcN8WGb0

---

## 📦 Repositório

**GitHub**

https://github.com/vrsm/togglemaster

---

## 👨‍🎓 Aluno

**Vinicius Matos**

RM370385

---

# 📑 Sumário

- Docker Compose
- Testes dos Microsserviços
- Kubernetes
- Escalabilidade do Evaluation Service
- Escalabilidade do Analytics Service
- DynamoDB
- Bancos de Dados
- Desafios
- Estrutura do Projeto

---

# 🐳 Parte 1 - Docker Compose (Ambiente Local)

## Subir todo o ambiente

```bash
docker compose up -d
```

---

## Verificar logs

```bash
docker compose logs --tail=20 | grep -Ei "stacktrace"
```

✅ Não deve existir nenhuma StackTrace.

---

# 🔐 Parte 2 - Testar Microsserviços

# Auth Service

## Health Check

```bash
curl http://localhost:8001/health
```

---

## Criar API Key

```bash
curl -X POST http://localhost:8001/admin/keys \
-H "Authorization: Bearer tm_key_14700ed377e62b253882073469888517a999a1a033a35074dade4519a0401d8b" \
-H "Content-Type: application/json" \
-d '{"name":"evaluation-service"}'
```

### Importante

Após gerar uma nova chave:

- Atualizar a chave no `docker-compose.yaml`
- Atualizar todos os tokens utilizados
- Reiniciar apenas o Evaluation Service

```bash
docker compose up -d --force-recreate evaluation-service
```

---

## Validar API Key

```bash
curl http://localhost:8001/validate \
-H "Authorization: Bearer SUA_API_KEY"
```

---

# 🚩 Flag Service

## Health

```bash
curl http://localhost:8002/health
```

---

## Criar Flag

```bash
curl -X POST \
http://localhost:8002/flags \
-H "Authorization: Bearer SUA_API_KEY" \
-H "Content-Type: application/json" \
-d '{
"name":"nova-home",
"description":"Nova Home",
"is_enabled":true
}'
```

---

## Buscar Flag

```bash
curl \
http://localhost:8002/flags/nova-home \
-H "Authorization: Bearer SUA_API_KEY"
```

---

## Validar PostgreSQL

```bash
docker exec -it flag-db psql -U postgres -d flags_db

select * from flags;
```

---

# 🎯 Targeting Service

## Health

```bash
curl http://localhost:8003/health
```

---

## Criar Regra

```bash
curl -X POST \
http://localhost:8003/rules \
-H "Authorization: Bearer SUA_API_KEY" \
-H "Content-Type: application/json" \
-d '{
"flag_name":"nova-home",
"rules":{
"type":"PERCENTAGE",
"value":50
}
}'
```

---

## Buscar Regra

```bash
curl \
http://localhost:8003/rules/nova-home \
-H "Authorization: Bearer SUA_API_KEY"
```

---

## Validar PostgreSQL

```sql
select * from targeting_rules;
```

---

# ⚙ Evaluation Service

## Health

```bash
curl http://localhost:8004/health
```

---

## Avaliar Flag

```bash
curl \
"http://localhost:8004/evaluate?user_id=user123&flag_name=nova-home"
```

---

## Testar Cache Redis

Abrir logs

```bash
docker compose logs -f evaluation-service
```

Limpar cache

```bash
docker exec -it redis redis-cli FLUSHALL
```

Executar duas vezes:

```bash
curl "http://localhost:8004/evaluate?user_id=user123&flag_name=nova-home"
```

---

# 📊 Analytics Service

## Health

```bash
curl http://localhost:8005/health
```

---

## Worker

```bash
docker compose logs analytics-service -f
```

---

## Gerar evento

```bash
curl \
"http://localhost:8004/evaluate?user_id=user123&flag_name=nova-home"
```

---

## Consultar DynamoDB Local

```bash
aws dynamodb scan \
--table-name toggle-master \
--endpoint-url http://localhost:8000
```

---

# ☸ Parte 3 - Kubernetes

## Cluster

```bash
kubectl get nodes
```

## Deployments

```bash
kubectl get deploy -A
```

## Pods

```bash
kubectl get pods -n togglemaster
```

## Services

```bash
kubectl get svc -n togglemaster
```

## Ingress

```bash
kubectl get ingress -n togglemaster
```

ou

```bash
kubectl get svc ingress-nginx-controller -n ingress-nginx
```

---

## Teste Público

```bash
curl \
http://LOAD_BALANCER/evaluate?user_id=user123&flag_name=nova-home
```

---

# 📈 Parte 4 - Escalabilidade do Evaluation Service

## Exibir HPA

```bash
kubectl get hpa -n togglemaster
```

---

## Gerar carga

```bash
hey -z 2m -c 100 \
http://LOAD_BALANCER/evaluate?user_id=user123&flag_name=nova-home
```

---

## Monitorar HPA

```bash
watch kubectl get hpa -n togglemaster
```

---

## Monitorar Pods

```bash
watch kubectl get pods -n togglemaster
```

---

# 📊 Parte 5 - Escalabilidade do Analytics Service

## Enviar mensagens para a SQS

Pode ser por script ou AWS CLI.

---

## Monitorar HPA

```bash
kubectl get hpa
```

---

## Monitorar Pods

```bash
kubectl get pods
```

---

# 🗄 Parte 6 - DynamoDB

Consultar eventos

```bash
aws dynamodb scan \
--table-name toggle-master
```

---

# 💾 Parte 7 - Bancos

## Auth

```sql
select * from api_keys;
```

---

## Flag

```sql
select * from flags;
```

---

## Targeting

```sql
select * from targeting_rules;
```

---

## Redis

```text
keys *
```

---

## DynamoDB

```text
scan
```

---

# ⚠ Parte 8 - Desafios

Durante o desenvolvimento foram enfrentados desafios relacionados a:

- Integração entre microsserviços
- IAM Role da AWS Academy
- Secrets no Kubernetes
- Configuração do HPA
- Autenticação entre Evaluation e Auth
- Permissões do DynamoDB
- Diferenças entre Docker Compose e Amazon EKS

---

# 📂 Parte 9 - Estrutura do Projeto

```
docker-compose.yaml

k8s/

auth-service/

flag-service/

targeting-service/

evaluation-service/

analytics-service/
```

---

# 🛠 Tecnologias Utilizadas

- Docker Compose
- Kubernetes
- Amazon EKS
- Amazon SQS
- Amazon DynamoDB
- PostgreSQL
- Redis
- Python
- Flask
- NGINX Ingress
- Horizontal Pod Autoscaler (HPA)

---

# 📄 Licença

Projeto desenvolvido para o **Tech Challenge - Pós-Tech FIAP**.
