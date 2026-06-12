# Ch01 — Instalação do Automation Dashboard

## O que é o Automation Dashboard

Aplicação containerizada separada do AAP que coleta métricas de execução de até **3 instâncias AAP da mesma versão** e exibe dashboards de savings, eficiência e operação. Ideal para ambientes que não podem enviar dados para console.redhat.com (air-gap).

## Pré-requisitos

| Requisito | Detalhe |
|---|---|
| **SO** | RHEL 9 ou RHEL 10 (x86_64 ou ARM) |
| **Hardware mínimo** | 4 vCPUs, 16 GB RAM, 80 GB disco, 3000 IOPS |
| **Banco** | PostgreSQL v15 externo (não no mesmo host do AAP) |
| **Capacidade** | Até 10.000 jobs/mês e 47M summaries/mês |
| **Porta de ingress** | 8447 (HTTPS, configurável) |
| **Conectividade** | HTTPS/443 bidirecional entre dashboard e instâncias AAP |

> **Importante:** Não instalar no mesmo host do AAP.

## Criar OAuth2 Application no AAP (antes da instalação)

```
AAP 2.5/2.6: https://AAP_GATEWAY_FQDN/access/applications
→ Add application:
    Name: automation-dashboard-sso
    Authorization grant type: authorization-code
    Organization: Default
    Redirect URI: https://DASHBOARD_FQDN/auth-callback
    Client type: Confidential
→ Salvar: client_id e client_secret
```

```
# Criar token de acesso (escopo: read)
AAP → Users → <usuário> → Tokens → Add token
→ OAuth application: automation-dashboard-sso
→ Scope: read
→ Salvar: access_token e refresh_token
```

## Inventory de Instalação

```ini
[automationdashboard]
dashboard.empresa.org ansible_connection=local

[automationdashboard:vars]
# OAuth2 para login no dashboard via AAP
aap_auth_provider_name=Ansible Automation Platform
aap_auth_provider_protocol=https
aap_auth_provider_aap_version=2.6
aap_auth_provider_host=gateway.empresa.org
aap_auth_provider_check_ssl=true
aap_auth_provider_client_id=<client_id>
aap_auth_provider_client_secret=<client_secret>

# Sincronização inicial
initial_sync_days=30     # importar últimos 30 dias de dados

# TLS customizado (opcional)
# dashboard_tls_cert=/path/to/dashboard.crt
# dashboard_tls_key=/path/to/dashboard.key

# Porta (padrão 8447)
# nginx_https_port=8447

[database]
dashboard.empresa.org ansible_connection=local

[redis]
dashboard.empresa.org ansible_connection=local

[all:vars]
redis_mode=standalone
postgresql_admin_username=postgres
postgresql_admin_password=SENHA_SEGURA

# Banco do dashboard
dashboard_pg_containerized=True
dashboard_admin_password=SENHA_ADMIN
dashboard_pg_host=db.empresa.org    # NÃO usar localhost ou 127.0.0.1
dashboard_pg_username=aapdashboard
dashboard_pg_password=SENHA_DB
dashboard_pg_database=aapdashboard

bundle_install=true
bundle_dir='{{ lookup("ansible.builtin.env", "PWD") }}/bundle'
```

## Instalação

```bash
# 1. Baixar installer do access.redhat.com
# Downloads → Red Hat Ansible Automation Platform Product Software
# → automation-dashboard-containerized-setup-bundle-*.tar.gz

# 2. Extrair
tar -xzvf ansible-automation-dashboard-containerized-setup-bundle-*.tar.gz
cd ansible-automation-dashboard-containerized-setup/

# 3. Instalar ansible-core
sudo dnf install ansible-core

# 4. Instalar coleções requeridas
ansible-galaxy collection install -r requirements.yml

# 5. Configurar inventory
cp -i inventory.example inventory
vi inventory   # preencher valores TODO

# 6. Executar instalação
ansible-playbook -i inventory \
  ansible.containerized_installer.dashboard_install \
  --ask-become-pass
```

## Verificar Containers em Execução

```bash
# Três containers devem estar rodando:
podman ps --all --format "{{.Names}}"
# postgresql
# automation-dashboard-task
# automation-dashboard-web
```
