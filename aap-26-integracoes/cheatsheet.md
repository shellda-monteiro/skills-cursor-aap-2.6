# Cheatsheet — Integrações, Hub e Controller

## External Secrets — Provedores

```
HashiCorp Vault KV    | HashiCorp Vault Secret Lookup
AWS Secrets Manager   | AWS Secrets Manager Lookup
Azure Key Vault       | Azure KMS Lookup
CyberArk CCP          | CyberArk Central Credential Provider Lookup
CyberArk Conjur       | CyberArk Conjur Secrets Manager Lookup
GitHub App Token      | GitHub App Installation Access Token Lookup
```

## hashicorp.vault — Auth

```bash
# Token auth
export VAULT_TOKEN=<token>
export VAULT_ADDR=https://vault.empresa.org

# AppRole auth
export VAULT_APPROLE_ROLE_ID=<role_id>
export VAULT_APPROLE_SECRET_ID=<secret_id>
```

## Private Hub — Workflow EE

```bash
podman login registry.redhat.io
podman pull registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest
podman tag <ee> hub.empresa.org/org/ee-supported:2.6
podman login hub.empresa.org
podman push hub.empresa.org/org/ee-supported:2.6
```

## Hub — Collections Sync

```
1. Token em console.redhat.com → Automation Hub → Connect to Hub
2. Hub UI → Remotes → rh-certified → Editar com token → Sync
3. Inventory: hub_seed_collections=true (requer 32GB RAM em growth)
```

## awx-manage

```bash
# Import inventory
sudo -u awx awx-manage inventory_import --source=/path/ --inventory-id=N

# Cleanup
sudo -u awx awx-manage cleanup_jobs --days=30
sudo -u awx awx-manage cleanup_activitystream --days=90

# Analytics
sudo -u awx awx-manage gather_analytics --ship
sudo -u awx awx-manage host_metric --since 2025-01-01 --json
```

## Backup OCP

```bash
# Criar backup via CRD
oc apply -f backup.yaml   # Kind: AnsibleAutomationPlatformBackup

# Restore via CRD
oc apply -f restore.yaml  # Kind: AnsibleAutomationPlatformRestore

# Must-gather
oc adm must-gather --image=registry.redhat.io/ansible-automation-platform-25/aap-must-gather-rhel8 --dest-dir=./gather
```
