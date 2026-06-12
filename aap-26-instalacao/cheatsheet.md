# Cheatsheet — AAP 2.6 Instalação

## Requisitos Mínimos por Topologia

| Topologia | VMs | RAM/VM | vCPU/VM | Disco | IOPS |
|---|---|---|---|---|---|
| Container Growth | 1 | 16 GB (32 GB c/ seed) | 4 | 60 GB total | **3000** |
| Container Enterprise | 4+ | 8–16 GB | 2–4 | 60 GB+ | **3000** |
| Operator Growth | OCP nodes | 8 GB | 2 | PVC via SC | — |
| Operator Enterprise | OCP nodes | 16 GB | 4 | PVC RWX / S3 | — |

## Versões Suportadas

| Software | Versão |
|---|---|
| RHEL | 9.4+ ou 10+ |
| ansible-core (RHEL 9) | 2.14 |
| ansible-core (RHEL 10) | 2.16 |
| PostgreSQL interno | 15 |
| PostgreSQL externo | 15, 16 ou 17 (com ICU) |
| OpenShift (Operator) | 4.x compatível com AAP 2.6 |
| Redis standalone | 1 nó |
| Redis cluster | 6 nós (3+3) |

## Portas Obrigatórias (resumo rápido)

| Porta | Protocolo | Serviço |
|---|---|---|
| **22** | SSH | Gerenciamento / installer |
| **80/443** | HTTP/HTTPS | UI e API |
| **5432** | PostgreSQL | Banco de dados |
| **6379** | Redis | Cache e filas |
| **16379** | Redis | Cluster bus (só HA) |
| **27199** | Receptor | Automation Mesh |
| **8443** | HTTPS | EDA event stream |

## Variáveis Críticas do Inventory (Containerizado)

```ini
# Autenticação no registry (online)
registry_username=<RHN_user>
registry_password=<RHN_password>

# Redis
redis_mode=standalone          # growth
# (omitir para cluster enterprise)

# Passwords obrigatórias
gateway_admin_password=<senha>
controller_admin_password=<senha>
hub_admin_password=<senha>
eda_admin_password=<senha>
postgresql_admin_password=<senha>

# Hosts do banco por componente
gateway_pg_host=<fqdn>
gateway_pg_password=<senha>
controller_pg_host=<fqdn>
controller_pg_password=<senha>
hub_pg_host=<fqdn>
hub_pg_password=<senha>
eda_pg_host=<fqdn>
eda_pg_password=<senha>

# Banco externo (PostgreSQL 15/16/17)
# NÃO incluir grupo [database] no inventory
automationcontroller_pg_database=awx
automationhub_pg_database=automationhub  # database DIFERENTE do Controller!

# TLS customizado
ca_tls_cert=/path/to/ca.crt
ca_tls_key=/path/to/ca.key
custom_ca_cert=/path/to/corporate-root-ca.crt

# Bundle/offline
bundle_install=true
bundle_dir=/path/to/bundle/
```

## Checklist Pré-Instalação

```
[ ] RHEL 9.4+ ou RHEL 10 com subscriptions ativas
[ ] Usuário não-root com sudo NOPASSWD configurado
[ ] ansible-core instalado (dnf install ansible-core)
[ ] NTP sincronizado em TODOS os nós
[ ] DNS: FQDN resolvível em todos os nós (sem underscore)
[ ] Podman home directory em disco LOCAL (não NFS!)
[ ] 3000 IOPS mínimo no volume do PostgreSQL
[ ] Portas 22, 443, 5432, 6379, 27199 liberadas no firewall
[ ] registry.redhat.io e cdn*.quay.io acessíveis (online)
[ ] registry_username/registry_password válidos
[ ] Certificados TLS em formato PEM com cadeia completa (se customizados)
[ ] Para Operator: namespace dedicado criado, StorageClass RWX disponível
[ ] Para DB externo: ICU habilitado, hstore habilitado no banco do Hub
```

## Comandos de Setup (Containerizado)

```bash
# Instalação
ansible-playbook -i inventory ansible.containerized_installer.install

# Com Vault e become interativo
ansible-playbook -i inventory -e @credentials.yml \
  --ask-vault-pass -K ansible.containerized_installer.install

# Backup
ansible-playbook -i inventory ansible.containerized_installer.backup

# Restore
ansible-playbook -i inventory ansible.containerized_installer.restore
ansible-playbook -i inventory ansible.containerized_installer.restore \
  -e "restore_backup_file=/path/backup.tar.gz"

# Uninstall
ansible-playbook -i inventory ansible.containerized_installer.uninstall

# Verificar inventory
ansible all -i inventory --list-hosts
```

## URLs de Firewall de Saída (obrigatórias)

```
registry.redhat.io:443
cdn.quay.io:443          # e cdn01 até cdn06
console.redhat.com:443
sso.redhat.com:443
automation-hub-prd.s3.amazonaws.com
galaxy.ansible.com:443
```

## OpenShift — Comandos Rápidos

```bash
# Status do AAP
oc get aap -n <namespace>
oc describe aap aap -n <namespace>

# URL do Gateway
oc get aap aap -n <namespace> -o jsonpath='{.status.gatewayURL}'

# Senha admin
oc get secret aap-admin-password -n <namespace> \
  -o jsonpath='{.data.password}' | base64 -d

# PVCs (Hub deve estar Bound)
oc get pvc -n <namespace>

# Logs do Operator
oc logs -n <namespace> deployment/aap-operator-controller-manager
```
