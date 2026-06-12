# Ch02 — Integração: clusters.yaml e Sincronização

## Arquivo clusters.yaml

O `clusters.yaml` define as instâncias AAP que o dashboard coleta:

```yaml
clusters:
  - protocol: https
    address: gateway.empresa.org       # IP ou FQDN sem http(s)://
    port: 443
    name: aap-producao                 # nome único
    access_token: <access_token>
    refresh_token: <refresh_token>
    client_id: <client_id>
    client_secret: <client_secret>
    verify_ssl: true                   # false para certificados self-signed
    sync_schedules:
      - name: "A cada 5 minutos"
        rrule: "DTSTART;TZID=America/Sao_Paulo:20250101T000000 FREQ=MINUTELY;INTERVAL=5"
        enabled: true

  # Segunda instância AAP (opcional, mesma versão)
  - protocol: https
    address: gateway-dr.empresa.org
    port: 443
    name: aap-disaster-recovery
    access_token: <access_token_dr>
    refresh_token: <refresh_token_dr>
    client_id: <client_id_dr>
    client_secret: <client_secret_dr>
    verify_ssl: true
```

> **Limite:** até 3 instâncias AAP da **mesma versão**.  
> **Certificados self-signed:** `verify_ssl: false` — o dashboard não suporta CA customizada.

## Carregar clusters.yaml no Container

```bash
# Copiar o arquivo para dentro do container
podman cp clusters.yaml automation-dashboard-web:/tmp/

# Carregar a configuração
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py setclusters /tmp/clusters.yaml

# O arquivo é removido automaticamente do /tmp após o carregamento
```

## Sincronizar Dados

```bash
# Modo interativo (pede confirmação)
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py syncdata

# Modo não-interativo (para scripts/cron)
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py syncdata \
  --since=2025-01-01 --until=2025-06-30
```

## Token Rotation

O dashboard rotaciona os tokens internamente mas **não atualiza** o `clusters.yaml` local.

```bash
# Recuperar tokens atuais válidos (para recriar o clusters.yaml)
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py getclusters --decrypt > clusters_current.yaml

# Usar output como novo input se necessário
podman cp clusters_current.yaml automation-dashboard-web:/tmp/
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py setclusters /tmp/clusters_current.yaml
```

> **Atenção:** o `refresh_token` só pode ser usado **uma vez**. Se expirar, criar novos tokens no AAP e recarregar.

## Verificar Status da Sincronização

```bash
# Logs do container de task (sincronização periódica)
journalctl CONTAINER_NAME=automation-dashboard-task -f

# Exemplo de log bem-sucedido:
# INFO Detected AAP version AAP 2.6 at https://gateway.empresa.org:443
# INFO Executing GET request to .../api/controller/v2/jobs/?...
# INFO Token refresh POST request succeeded with status 201
```
