# Ch05 — API REST e Webhooks

## Endpoints Base da API (AAP 2.6)

| Componente | Endpoint base |
|---|---|
| Platform Gateway | `https://<gateway>/api/gateway/v1/` |
| Automation Controller | `https://<gateway>/api/controller/v2/` |
| Automation Hub | `https://<gateway>/api/hub/v3/` |
| EDA Controller | `https://<gateway>/api/eda/v1/` |

> **Mudança no AAP 2.6:** Todos os componentes são acessados via Platform Gateway. URLs diretas do Controller ainda funcionam para compatibilidade com 2.4.

## Métodos de Autenticação na API

### 1. OAuth 2 Token (recomendado para integração)

```bash
# Criar token (POST com Basic Auth)
curl -u admin:senha -k -X POST \
  https://gateway.empresa.org/api/gateway/v1/tokens/

# Resposta: {"token": "abc123...", "expires": "..."}

# Usar token
curl -k -X GET \
  -H "Authorization: Bearer abc123..." \
  https://gateway.empresa.org/api/controller/v2/job_templates/
```

Características: timeout configurável, revogável, com escopo (`read` ou `write`).

### 2. Basic Auth (para scripts simples ou testes)

```bash
curl -k --user admin:senha \
  https://gateway.empresa.org/api/controller/v2/hosts/
```

> Para desabilitar Basic Auth por segurança: Platform Gateway → Settings → Gateway basic auth enabled: OFF

### 3. Session Auth (browser/UI apenas)

```bash
# Passo 1: obter CSRF token
curl -k -c cookies.txt \
  https://gateway.empresa.org/api/gateway/v1/login/

# Passo 2: login
curl -X POST \
  -H "X-CSRFToken: <token>" \
  --cookie "csrftoken=<token>" \
  --data "username=admin&password=senha" \
  https://gateway.empresa.org/api/gateway/v1/login/
```

## Operações Comuns via API

### Disparar Job Template

```bash
# Launch simples
curl -k -X POST \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  https://gateway.empresa.org/api/controller/v2/job_templates/14/launch/

# Com extra_vars e limit
curl -k -X POST \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"limit": "web1.empresa.org", "extra_vars": {"version": "2.1.0"}}' \
  https://gateway.empresa.org/api/controller/v2/job_templates/14/launch/
```

### Verificar Status do Job

```bash
# Status do job (substituir <job_id>)
curl -k -H "Authorization: Bearer <token>" \
  https://gateway.empresa.org/api/controller/v2/jobs/<job_id>/

# Output do job (stdout)
curl -k -H "Authorization: Bearer <token>" \
  https://gateway.empresa.org/api/controller/v2/jobs/<job_id>/stdout/?format=txt
```

### Filtrar e Pesquisar

```bash
# Jobs com falha nas últimas 24h
GET /api/controller/v2/jobs/?status=failed&created__gt=2025-01-01

# Job Templates de uma organização
GET /api/controller/v2/job_templates/?organization__name=TI-Infraestrutura

# Hosts de um inventory com facts
GET /api/v2/hosts/?inventory=5&search=linux

# Sliced jobs
GET /api/controller/v2/jobs/?job_slice_count__gt=1
```

### Paginação

```bash
# Padrão: 25 itens por página
GET /api/controller/v2/hosts/?page_size=100&page=2
```

### Ordenação

```bash
# Ordenar por data de criação (decrescente)
GET /api/controller/v2/jobs/?order_by=-created

# Crescente
GET /api/controller/v2/jobs/?order_by=created
```

## Webhooks — Integrar CI/CD com o Controller

### GitHub/GitLab → AAP

```
Job Template ou Workflow Template → Enable Webhook
→ Webhook Service: GitHub (ou GitLab)
→ Webhook URL: https://gateway.empresa.org/api/controller/v2/job_templates/5/github/
→ Webhook Key: (copiar e configurar no GitHub → Settings → Webhooks)
```

**No GitHub:**
```
Repository → Settings → Webhooks → Add webhook
→ Payload URL: <Webhook URL do AAP>
→ Content type: application/json
→ Secret: <Webhook Key do AAP>
→ Events: Push events (ou Pull request)
```

### GitLab → AAP

```
Project → Settings → Webhooks
→ URL: <Webhook URL do AAP>
→ Secret Token: <Webhook Key do AAP>
→ Trigger: Push events
```

### Verificar Payload do Webhook Recebido

```bash
# Listar webhooks recebidos
GET /api/controller/v2/job_templates/<id>/webhook_receiver/

# O payload do webhook fica disponível como extra_var: awx_webhook_payload
- name: Usar dados do webhook
  debug:
    msg: "Branch: {{ awx_webhook_payload.ref | regex_replace('refs/heads/', '') }}"
```

## Exemplo Completo: Pipeline CI/CD

```bash
#!/bin/bash
# script de CI: após build bem-sucedido, disparar deploy no AAP

AAP_URL="https://gateway.empresa.org"
TOKEN="seu_token_oauth2"
TEMPLATE_ID=42
VERSION="${CI_COMMIT_SHORT_SHA}"
ENV="${DEPLOY_ENV:-staging}"

RESPONSE=$(curl -s -k -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"extra_vars\": {\"app_version\": \"$VERSION\", \"ambiente\": \"$ENV\"}}" \
  "$AAP_URL/api/controller/v2/job_templates/$TEMPLATE_ID/launch/")

JOB_ID=$(echo $RESPONSE | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")
echo "Job disparado: ID=$JOB_ID"
echo "$AAP_URL/#/jobs/playbook/$JOB_ID"
```
