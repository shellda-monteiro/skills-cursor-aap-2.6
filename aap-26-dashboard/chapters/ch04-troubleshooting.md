# Ch04 — Troubleshooting

## Erros de Autenticação e Sincronização

| Erro | Causa provável | Solução |
|---|---|---|
| **401** | Token expirado ou inválido | Verificar `access_token` no `clusters.yaml`; se expirado, criar novos tokens no AAP e recarregar |
| **401 transitório** | Token em rotação (comportamento esperado) | Aguardar — o dashboard tenta refresh automaticamente após 401 |
| **401 persistente** | `client_secret` ou `refresh_token` inválido | Executar `getclusters --decrypt` e comparar com o `clusters.yaml` original |
| **404** | URL do AAP incorreta | Verificar `address` e `port` no `clusters.yaml` |

> **Atenção:** o `refresh_token` pode ser usado **apenas uma vez**. Após a rotação interna, ele expira. Se precisar recarregar o `clusters.yaml`, obter tokens frescos primeiro com `getclusters --decrypt`.

## Comandos de Diagnóstico

```bash
# Ver tokens armazenados (descriptografados)
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py getclusters --decrypt

# Ver logs de sincronização em tempo real
journalctl CONTAINER_NAME=automation-dashboard-task -f

# Verificar status dos containers
podman ps --all --format "{{.Names}} {{.Status}}"

# Verificar status via systemd
systemctl status --user automation-dashboard-task
systemctl status --user automation-dashboard-web
systemctl status --user postgresql

# Logs de um container específico
journalctl CONTAINER_NAME=automation-dashboard-web -n 100
```

## Jobs de Sincronização Travados (após upgrade/restart)

**Sintoma:** novos jobs do AAP não aparecem no dashboard após upgrade ou restart do serviço.

```bash
# 1. Conectar ao banco de dados
PGPASSWORD=<dashboard_pg_password> psql \
  -h 127.0.0.1 -p 5432 \
  -U aapdashboard -d aapdashboard

# 2. Identificar jobs presos
SELECT * FROM scheduler_syncjob WHERE status IN ('pending','waiting','running');

# 3. Aguardar ~1 minuto e rodar novamente
# Se os mesmos IDs aparecerem, estão travados

# 4. Deletar os jobs travados (substituir <id> pelos IDs encontrados)
DELETE FROM scheduler_syncjob WHERE id IN (<id1>, <id2>);

# 5. Verificar no dashboard se a sincronização retomou
```

## Log de Sincronização Bem-sucedida

```
INFO Detected AAP version AAP 2.6 at https://gateway.empresa.org:443
INFO Executing GET request to .../api/controller/v2/organizations/?...
ERROR GET request failed with status 401          ← token expirado (normal)
INFO Token refresh POST request succeeded 201     ← renovação automática
INFO Executing GET request to .../api/controller/v2/job_templates/?...
INFO Executing GET request to .../api/controller/v2/jobs/?...
INFO Successfully created Sync task for Cluster gateway.empresa.org
```

## Checklist de Verificação Pós-instalação

```
✓ podman ps mostra: postgresql, automation-dashboard-task, automation-dashboard-web
✓ Acesso à UI: https://dashboard.empresa.org:8447
✓ Login via AAP OAuth2 funciona
✓ clusters.yaml carregado (setclusters sem erro)
✓ syncdata manual executa sem 404
✓ Dashboard exibe jobs do AAP após sync
✓ Filtros de Template/Organization funcionam
```
