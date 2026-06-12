# Ch03 — Instalação Containerizada (Passo a Passo)

## Fluxo de Instalação

```
1. Preparar RHEL host (usuário, sudo, NTP, DNS, ansible-core)
2. Baixar installer (.tar.gz online ou bundle offline)
3. Editar inventory file
4. Executar ansible-playbook -i <inventory> ansible.containerized_installer.install
5. Ativar subscrição (credenciais ou manifest file)
```

## Tipos de Instalação

| Tipo | Quando | registry_username/password | bundle_install |
|---|---|---|---|
| **Online** | Host com acesso à internet | **Obrigatório** | false (padrão) |
| **Offline/Bundle** | Ambiente air-gapped | Não necessário | `true` + `bundle_dir` |

### Download
```bash
# Online installer
wget https://access.redhat.com/.../ansible-automation-platform-containerized-setup-<version>.tar.gz

# Bundle (offline)
wget https://access.redhat.com/.../ansible-automation-platform-containerized-setup-bundle-<version>-x86_64.tar.gz

# Extrair (mínimo 15 GB no diretório de instalação)
tar xfvz ansible-automation-platform-containerized-setup-<version>.tar.gz
cd ansible-automation-platform-containerized-setup-<version>/
```

## Estrutura do Instalador

```
ansible-automation-platform-containerized-setup-<version>/
├── inventory             ← inventory enterprise (padrão)
├── inventory-growth      ← inventory growth (all-in-one)
├── README.md             ← documentação de variáveis
└── collections/          ← collection ansible.containerized_installer empacotada
```

**Usar `inventory-growth` para topologia all-in-one; `inventory` para enterprise.**

## Executar Instalação

```bash
# Instalação padrão (online)
ansible-playbook -i inventory ansible.containerized_installer.install

# Com senhas em Vault e privilege escalation interativo
ansible-playbook -i inventory -e @credentials.yml \
  --ask-vault-pass -K -v ansible.containerized_installer.install

# Parâmetros opcionais:
#   -i <inventory>        inventory a usar
#   -e @<vault_file>      variáveis sensíveis via Ansible Vault
#   --ask-vault-pass       solicita senha do Vault
#   -K                     solicita senha de become (sudo)
#   -v / -vv / -vvv / -vvvv  verbosidade (aumenta tempo de execução)
```

**Nota:** A instalação containerizada AAP 2.6 usa playbooks da collection `ansible.containerized_installer` via `ansible-playbook`. O script `setup.sh`/`aap_setup.sh` pertence ao instalador RPM (deprecated), não ao containerizado.

## Segurança de Senhas no Inventory

```bash
# Criar arquivo separado para credenciais
cat > credentials.yml << EOF
gateway_admin_password: my_secure_password
postgresql_admin_password: my_pg_password
registry_password: my_registry_password
EOF

# Encriptar com Ansible Vault
ansible-vault encrypt credentials.yml

# Executar instalação com vault
ansible-playbook -i inventory -e @credentials.yml \
  --ask-vault-pass -K ansible.containerized_installer.install
```

**NUNCA versionar inventory com senhas em texto claro no Git.**

## Configuração de PostgreSQL Externo

```ini
[all:vars]
# Desabilitar banco interno — NÃO incluir grupo [database] no inventory

# Credenciais admin do PostgreSQL (para criar databases)
postgresql_admin_username=postgres
postgresql_admin_password=<admin_senha>

# Controller DB
controller_pg_host=db.example.org
controller_pg_port=5432
controller_pg_database=awx
controller_pg_username=awx
controller_pg_password=<senha>

# Hub DB (database diferente do Controller!)
hub_pg_host=db.example.org
hub_pg_database=automationhub
hub_pg_username=automationhub
hub_pg_password=<senha>

# Gateway DB
gateway_pg_host=db.example.org
gateway_pg_database=automationgateway
gateway_pg_password=<senha>

# EDA DB
eda_pg_host=db.example.org
eda_pg_database=automationeda
eda_pg_password=<senha>
```

### Habilitar hstore para Hub
```sql
-- Conectar ao banco automationhub como superuser
\c automationhub
CREATE EXTENSION IF NOT EXISTS hstore;
```

## Configuração de Certificados TLS Customizados

### Opção 1: CA customizada gera todos os certificados
```ini
[all:vars]
ca_tls_cert=/path/to/ca.crt
ca_tls_key=/path/to/ca.key
```

### Opção 2: Certificados individuais por serviço
```ini
[all:vars]
# Gateway
gateway_tls_cert=/path/to/gateway.crt
gateway_tls_key=/path/to/gateway.key

# Controller
controller_tls_cert=/path/to/controller.crt
controller_tls_key=/path/to/controller.key

# Hub
hub_tls_cert=/path/to/hub.crt
hub_tls_key=/path/to/hub.key

# EDA
eda_tls_cert=/path/to/eda.crt
eda_tls_key=/path/to/eda.key
```

**Certificados devem estar em formato PEM com a cadeia completa da CA Raiz.**

## Storage para Automation Hub

### NFS (suportado para Hub — NÃO para containers Podman)
```ini
[all:vars]
hub_storage_backend=file
hub_shared_data_path=/mnt/nfs/hub-data
```

### Amazon S3
```ini
[all:vars]
hub_storage_backend=s3
hub_s3_bucket=my-aap-bucket
hub_s3_region=us-east-1
hub_s3_access_key=<access_key>
hub_s3_secret_key=<secret_key>
```

### Azure Blob
```ini
[all:vars]
hub_storage_backend=azure
hub_azure_account_name=<storage_account>
hub_azure_account_key=<account_key>
hub_azure_container=aap-hub
```

## Instalação Desconectada (Air-Gapped)

```bash
# Configurar repositório local a partir de ISO
sudo mount -o loop rhel-9.x-x86_64-dvd.iso /mnt/iso
sudo dnf config-manager --add-repo file:///mnt/iso/BaseOS
sudo dnf config-manager --add-repo file:///mnt/iso/AppStream

# Ou usando reposync para mirror
sudo reposync --repo=rhel-9-for-x86_64-baseos-rpms --download-path=/var/repos

# Inventory para instalação bundle
bundle_install=true
bundle_dir='{{ lookup("ansible.builtin.env", "PWD") }}/bundle'
# NÃO incluir registry_username/registry_password
```

## Manutenção

```bash
# Upgrade — re-executar install com o installer da mesma versão instalada
ansible-playbook -i inventory ansible.containerized_installer.install

# Backup (destino padrão: ~/backups; customizar com backup_dir no inventory)
ansible-playbook -i inventory ansible.containerized_installer.backup

# Restore (mesmo ambiente/hostnames)
ansible-playbook -i inventory ansible.containerized_installer.restore

# Restore com arquivo específico
ansible-playbook -i inventory ansible.containerized_installer.restore \
  -e "restore_backup_file=/path/to/backup.tar.gz"

# Uninstall
ansible-playbook -i inventory ansible.containerized_installer.uninstall

# Uninstall preservando imagens ou bancos
ansible-playbook -i inventory ansible.containerized_installer.uninstall \
  -e container_keep_images=true
ansible-playbook -i inventory ansible.containerized_installer.uninstall \
  -e postgresql_keep_databases=true

# Reinstall preservando banco (coletar secret keys com podman secret inspect antes)
ansible-playbook -i inventory ansible.containerized_installer.install \
  -e controller_secret_key=<secret_key_value>
```
