# Ch05 — Controller Avançado: Secrets, awx-manage e Backup OCP

## External Secret Management — Provedores Suportados

O Automation Controller integra com os seguintes provedores de secrets externos:

| Provedor | Tipo de Credential |
|---|---|
| **HashiCorp Vault KV** | HashiCorp Vault Secret Lookup |
| **HashiCorp Vault SSH** | HashiCorp Vault Signed SSH |
| **AWS Secrets Manager** | AWS Secrets Manager Lookup |
| **Microsoft Azure Key Vault** | Azure Key Management System |
| **CyberArk CCP** | CyberArk Central Credential Provider Lookup |
| **CyberArk Conjur** | CyberArk Conjur Secrets Manager Lookup |
| **Centrify Vault** | Centrify Vault Credential Provider Lookup |
| **Thycotic DevOps Vault** | Thycotic DevOps Secrets Vault |
| **Thycotic Secret Server** | Thycotic Secret Server |
| **GitHub App Token** | GitHub App Installation Access Token Lookup |

### Configurar Secret Lookup (exemplo HashiCorp Vault)

```
1. Controller UI → Automation Execution → Infrastructure → Credentials → Add
2. Tipo: HashiCorp Vault Secret Lookup
3. Campos: Vault Address, Token (ou AppRole)

4. Na Credential de destino (ex: Machine Credential):
   → Password: clicar no ícone de chave → selecionar Vault Lookup
   → Metadata: informar path do segredo no Vault
```

### Metadata por provedor

| Provedor | Metadados necessários |
|---|---|
| AWS Secrets Manager | Region (obrigatório), Secret Name (obrigatório) |
| Centrify Vault | Account name (obrigatório), System Name |
| CyberArk CCP | Object Query (obrigatório), App ID, Policy ID |
| HashiCorp Vault KV | Secret Path (obrigatório), API Version (v1/v2) |

## Security Best Practices do Controller

### Princípios

1. **Não expor o Controller na internet** — acesso apenas via rede interna/VPN
2. **Mínimo de contas administrativas** — não conceder sudo/root ao usuário `awx` para não-admins
3. **RBAC rigoroso** — delegar mínimo de privilégios por role
4. **Times vs usuários individuais** — atribuir permissões a times, não diretamente a usuários
5. **Separação de responsabilidades** — credenciais diferentes para níveis diferentes de acesso
6. **Não desabilitar SELinux** nem o containment multi-tenant do Controller
7. **Source control obrigatório** — toda automação em Git com code review

### Isolamento de Credenciais

```yaml
# Credenciais por nível de acesso:
# - patching: acesso sudo para aplicar patches
# - app-deploy: acesso para deploy de aplicações
# - readonly: apenas verificação/auditoria

# Job Template de patching usa credential "patching-key"
# Job Template de deploy usa credential "app-deploy-key"
# Nenhum usuário vê as chaves privadas diretamente
```

## awx-manage — Utilitário de Linha de Comando

> **IMPORTANTE:** Executar como usuário `awx` apenas. Não rodar outros comandos sem orientação do suporte Ansible.

### Import de Inventory

```bash
# Importar inventory de arquivo
sudo -u awx awx-manage inventory_import \
  --source=/ansible/inventories/producao/ \
  --inventory-id=5

# Sobrescrever dados existentes completamente
sudo -u awx awx-manage inventory_import \
  --source=/ansible/inventories/producao/ \
  --inventory-id=5 \
  --overwrite

# Sobrescrever variáveis de hosts (preservar novos hosts)
sudo -u awx awx-manage inventory_import \
  --source=/ansible/inventories/producao/ \
  --inventory-id=5 \
  --overwrite_vars
```

### Limpeza de Dados Antigos

```bash
# Ver opções de limpeza de jobs
sudo -u awx awx-manage cleanup_jobs --help

# Limpar jobs mais antigos que N dias
sudo -u awx awx-manage cleanup_jobs --days=30

# Limpar Activity Stream antigo
sudo -u awx awx-manage cleanup_activitystream --days=90
```

### Analytics

```bash
# Coletar analytics imediatamente (sem esperar a janela de 4h)
sudo -u awx awx-manage gather_analytics --ship

# Ambientes desconectados: coletar métricas de hosts únicos automatizados
sudo -u awx awx-manage host_metric --since 2025-01-01 --json
```

## Backup e Restore — Containerizado (VM-based)

```bash
# Backup
cd /path/to/aap-installer/
ansible-playbook -i inventory ansible.containerized_installer.backup

# Restore
ansible-playbook -i inventory ansible.containerized_installer.restore \
  -e "restore_backup_file=/path/to/backup.tar.gz"
```

## Backup e Restore — OpenShift Operator

### Backup via AnsibleAutomationPlatformBackup CRD

```yaml
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformBackup
metadata:
  name: aap-backup-2025-01
  namespace: aap
spec:
  deploymentName: aap
  backupPVC: aap-backup-pvc
  backupPVCStorageRequirements: "20Gi"
```

### Restore via AnsibleAutomationPlatformRestore CRD

```yaml
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformRestore
metadata:
  name: aap-restore
  namespace: aap
spec:
  deploymentName: aap
  backupName: aap-backup-2025-01
```

### Verificar status do backup

```bash
oc get ansibleautomationplatformbackup -n aap
oc describe ansibleautomationplatformbackup aap-backup-2025-01 -n aap
```

### Coletar dados do OCP (must-gather)

```bash
oc adm must-gather \
  --image=registry.redhat.io/ansible-automation-platform-25/aap-must-gather-rhel8 \
  --dest-dir=./aap-gather

# Sem acesso ao registry.redhat.io:
oc adm inspect
```
