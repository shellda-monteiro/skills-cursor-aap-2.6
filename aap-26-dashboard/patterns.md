# Patterns — Automation Dashboard

## Pattern 1: Dashboard Multi-cluster (Produção + DR)

**Cenário:** Monitorar duas instâncias AAP (produção e disaster recovery) em um único dashboard.

```yaml
# clusters.yaml
clusters:
  - name: aap-producao
    protocol: https
    address: gateway-prod.empresa.org
    port: 443
    access_token: <token_prod>
    refresh_token: <refresh_prod>
    client_id: <id_prod>
    client_secret: <secret_prod>
    verify_ssl: true
    sync_schedules:
      - name: "Sync 5min"
        rrule: "DTSTART;TZID=America/Sao_Paulo:20250101T000000 FREQ=MINUTELY;INTERVAL=5"
        enabled: true

  - name: aap-dr
    protocol: https
    address: gateway-dr.empresa.org
    port: 443
    access_token: <token_dr>
    refresh_token: <refresh_dr>
    client_id: <id_dr>
    client_secret: <secret_dr>
    verify_ssl: true
    sync_schedules:
      - name: "Sync 15min"
        rrule: "DTSTART;TZID=America/Sao_Paulo:20250101T000000 FREQ=MINUTELY;INTERVAL=15"
        enabled: true
```

---

## Pattern 2: Relatório Mensal de ROI para Gestor

```
1. Dashboard → Reports → Filters:
   - Time period: último mês completo
   - Organization: todas

2. Cost and Savings Analysis:
   - Monthly AAP cost: R$ 8.000 (licença + infra)
   - Hourly rate: R$ 120 (custo/hora analista)
   - Time to create automation: habilitado

3. Verificar:
   - Total savings → quanto foi poupado no mês
   - Top 5 projects → quais projetos geram mais savings
   - Failed jobs → taxa de falha → meta < 5%

4. Export → Save as PDF
   → Enviar para gestor com o relatório gerado
```

---

## Pattern 3: Cron de Sincronização Manual (Complementar)

```bash
# Para ambientes onde o schedule interno não é suficiente
# /etc/cron.d/aap-dashboard-sync
0 */4 * * * usuario podman exec automation-dashboard-web \
  /venv/bin/python manage.py syncdata \
  --since=$(date -d '8 hours ago' +%Y-%m-%dT%H:%M:%S) \
  --until=$(date +%Y-%m-%dT%H:%M:%S) >> /var/log/dashboard-sync.log 2>&1
```

---

## Pattern 4: Recuperar Acesso após Expiração de Tokens

```bash
# 1. Criar novos tokens no AAP (access + refresh)
# AAP → Users → <usuário> → Tokens → Add token (scope: read)

# 2. Atualizar clusters.yaml com novos tokens

# 3. Recarregar
podman cp clusters.yaml automation-dashboard-web:/tmp/
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py setclusters /tmp/clusters.yaml

# 4. Verificar tokens armazenados
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py getclusters --decrypt

# 5. Sincronizar dados do período perdido
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py syncdata --since=2025-01-01 --until=2025-12-31
```
