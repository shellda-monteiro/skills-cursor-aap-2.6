# Cheatsheet — Automation Dashboard

## Instalação

```bash
ansible-playbook -i inventory \
  ansible.containerized_installer.dashboard_install --ask-become-pass
```

## Containers em execução

```bash
podman ps --format "{{.Names}}"
# postgresql | automation-dashboard-task | automation-dashboard-web
```

## clusters.yaml — Estrutura mínima

```yaml
clusters:
  - protocol: https
    address: gateway.empresa.org
    port: 443
    name: producao
    access_token: <token>
    refresh_token: <refresh>
    client_id: <id>
    client_secret: <secret>
    verify_ssl: true
```

## Comandos essenciais

```bash
# Carregar clusters
podman cp clusters.yaml automation-dashboard-web:/tmp/
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py setclusters /tmp/clusters.yaml

# Sincronizar dados (intervalo manual)
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py syncdata --since=2025-01-01 --until=2025-12-31

# Ver tokens atuais
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py getclusters --decrypt

# Logs em tempo real
journalctl CONTAINER_NAME=automation-dashboard-task -f
```

## Troubleshooting rápido

| Erro | Ação |
|---|---|
| 401 persistente | `getclusters --decrypt` → comparar tokens → criar novos tokens no AAP |
| 404 | Verificar `address` e `port` no clusters.yaml |
| Jobs parados | `DELETE FROM scheduler_syncjob WHERE status IN ('pending','running')` |
| Dashboard vazio | Verificar `initial_sync_days` no inventory e rodar syncdata |

## Diferença Dashboard vs Analytics

| | Dashboard (local) | Analytics (cloud) |
|---|---|---|
| Instalação | Servidor RHEL dedicado | SaaS (console.redhat.com) |
| Air-gap | ✓ Sim | ✗ Não |
| Instâncias conectadas | Até 3 (mesma versão) | Ilimitado |
| Exportar CSV/PDF | ✓ Sim | ✓ Sim |
