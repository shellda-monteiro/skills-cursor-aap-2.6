# Ch05 — Operações de Dia-2

## Backup e Restore (Containerizado)

```bash
# Backup completo (destino padrão: ~/backups)
ansible-playbook -i inventory ansible.containerized_installer.backup

# Backup com destino customizado
ansible-playbook -i inventory ansible.containerized_installer.backup \
  -e "backup_dir=/mnt/backup/aap/"

# Restore (mesmo ambiente/hostnames)
ansible-playbook -i inventory ansible.containerized_installer.restore

# Restore com arquivo específico
ansible-playbook -i inventory ansible.containerized_installer.restore \
  -e "restore_backup_file=/mnt/backup/aap/aap-backup-<timestamp>.tar.gz"

# O arquivo de backup inclui:
# - PostgreSQL (todos os bancos: controller, hub, eda, gateway)
# - Configurações de Redis
# - Certificados e chaves
# - Configuração do Receptor
```

## Backup e Restore (OpenShift Operator)

```bash
# Criar AnsibleAutomationPlatformBackup CR
cat <<EOF | oc apply -f -
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformBackup
metadata:
  name: aap-backup-$(date +%Y%m%d)
  namespace: aap-production
spec:
  no_log: false
  deployment_name: aap
  pvc_claim_name: aap-backup-storage
EOF

# Verificar status
oc get aapbackup -n aap-production
oc describe aapbackup aap-backup-<data> -n aap-production

# Restore
cat <<EOF | oc apply -f -
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformRestore
metadata:
  name: aap-restore
  namespace: aap-production
spec:
  deployment_name: aap
  backup_name: aap-backup-<data>
EOF
```

## Upgrade (Containerizado)

```bash
# 1. Fazer backup ANTES do upgrade
ansible-playbook -i inventory ansible.containerized_installer.backup

# 2. Baixar nova versão do installer
tar xfvz ansible-automation-platform-containerized-setup-<nova-versao>.tar.gz

# 3. Copiar inventory customizado para o novo diretório
cp /path/to/my-inventory ansible-automation-platform-containerized-setup-<nova-versao>/inventory

# 4. Executar upgrade (mesmo comando de instalação)
cd ansible-automation-platform-containerized-setup-<nova-versao>/
ansible-playbook -i inventory ansible.containerized_installer.install

# O installer detecta instalação existente e faz upgrade preservando dados
```

## Scaling Horizontal (Containerizado Enterprise)

### Adicionar Execution Nodes
```ini
# No inventory, adicionar novo nó
[execution_nodes]
exec1.exemplo.org
exec2.exemplo.org  # novo nó adicionado

[execution_nodes:vars]
node_type=execution
```
```bash
# Re-executar instalação para provisionar apenas o novo nó
ansible-playbook -i inventory ansible.containerized_installer.install \
  --limit exec2.exemplo.org
```

### Adicionar EDA Controller (enterprise)
```ini
[automationeda]
eda1.exemplo.org
eda2.exemplo.org  # novo nó EDA
```

## Gestão de Collections no Private Hub

```bash
# Sincronizar collections certificadas da Red Hat
# Hub UI → Collections → Repositories → Sync

# Upload de collection customizada
ansible-galaxy collection build
ansible-galaxy collection publish <collection>.tar.gz \
  --server https://hub.exemplo.org/api/galaxy/ \
  --token <hub-token>

# Aprovação de collections (se aprovação habilitada)
# Hub UI → Collections → Needs Review → Approve
```

## Gestão de Assinaturas (Subscriptions)

```bash
# Verificar subscription ativa
# Controller UI → Settings → Subscription

# Atualizar manifest file
# Controller UI → Settings → Subscription → Browse → Upload manifest.zip

# Gerar manifest file
# https://access.redhat.com/management → Subscriptions → Ansible Automation Platform
# → Create Allocation → Download Manifest
```

## Monitoramento e Analytics

```bash
# Habilitar envio de dados para Red Hat Insights
# Controller UI → Settings → Miscellaneous System Settings
# INSIGHTS_TRACKING_STATE: true

# Acessar Automation Analytics
# https://console.redhat.com/ansible/automation-analytics

# Métricas locais via API
GET /api/controller/v2/dashboard/
GET /api/controller/v2/metrics/    # métricas Prometheus
```

## Reinicialização Controlada (Containerizado)

```bash
# Parar todos os serviços AAP
podman pod stop --all  # ou por componente

# Iniciar na ordem correta
podman start automation-postgresql
sleep 10
podman start automation-redis
sleep 5
podman start automation-controller
podman start automation-hub
podman start automation-eda-server
podman start automation-gateway

# Verificar saúde
podman ps
curl -k https://gateway.exemplo.org/api/gateway/v1/ping/
```
