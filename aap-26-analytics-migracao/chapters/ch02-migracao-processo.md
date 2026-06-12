# Ch02 — Processo de Migração: Passo a Passo

## Estrutura do Migration Artifact

```
artifact/
├── controller/
│   ├── controller.pgc          # dump do banco do Controller
│   └── custom_configs/         # configurações customizadas
├── gateway/
│   └── gateway.pgc             # dump do banco do Gateway
├── hub/
│   └── hub.pgc                 # dump do banco do Hub
├── secrets.yml                 # SECRET_KEYs + hub_db_fields_encryption_key
└── sha256sum.txt               # checksum de todos os arquivos .pgc
```

## ETAPA 1: Exportar o Ambiente Fonte (RPM como exemplo)

### 1.1 Verificar conectividade com os bancos

```bash
# Verificar banco do Controller
psql -h <pg_hostname> -U <controller_pg_user> -d <database_name> \
  -t -c 'SHOW server_version;'

# Repetir para gateway e hub
```

### 1.2 Criar estrutura de diretórios

```bash
mkdir -p /tmp/backups/artifact/{controller,gateway,hub}
mkdir -p /tmp/backups/artifact/controller/custom_configs
touch /tmp/backups/artifact/secrets.yml
cd /tmp/backups/artifact/
```

### 1.3 Realizar dumps dos bancos

```bash
# Controller
pg_dump -h <pg_hostname> -U <controller_pg_user> -d <controller_pg_name> \
  --clean --create -Fc -f controller/controller.pgc

# Gateway
pg_dump -h <pg_hostname> -U <gateway_pg_user> -d <gateway_pg_name> \
  --clean --create -Fc -f gateway/gateway.pgc

# Hub
pg_dump -h <pg_hostname> -U <hub_pg_user> -d <hub_pg_name> \
  --clean --create -Fc -f hub/hub.pgc

# Verificar arquivos gerados
ls -ld controller/controller.pgc gateway/gateway.pgc hub/hub.pgc
```

### 1.4 Coletar SECRET_KEYs (RPM)

```bash
# Controller
cat /etc/tower/SECRET_KEY
# Adicionar ao secrets.yml:
echo "controller_secret_key: <valor>" >> secrets.yml

# Hub
grep '^SECRET_KEY' /etc/pulp/settings.py | awk -F'=' '{ print $2 }'
echo "hub_secret_key: <valor>" >> secrets.yml

# Hub database fields encryption key
cat /etc/pulp/certs/database_fields.symmetric.key
echo "hub_db_fields_encryption_key: <valor>" >> secrets.yml

# Gateway
cat /etc/ansible-automation-platform/gateway/SECRET_KEY
echo "gateway_secret_key: <valor>" >> secrets.yml
```

### 1.5 Coletar SECRET_KEYs (Container-based)

```bash
# Usar podman secret ou variável de ambiente do container
# Controller
podman exec automation-controller \
  /usr/bin/awx-manage print_settings | grep '^SECRET_KEY'

# Gateway
podman exec automation-gateway \
  /usr/bin/aap-gateway-manage print_settings | grep '^SECRET_KEY'

# Hub
grep '^SECRET_KEY' /etc/pulp/settings.py
```

### 1.6 Criar e verificar o artifact tar

```bash
cd /tmp/backups/artifact/

# Criar checksum
[ -f sha256sum.txt ] && rm -f sha256sum.txt
find . -type f -name "*.pgc" -exec sha256sum {} \; >> sha256sum.txt
cat sha256sum.txt

# Criar tarball
cd ..
tar cf artifact.tar artifact
sha256sum artifact.tar > artifact.tar.sha256
sha256sum --check artifact.tar.sha256

# Verificar conteúdo
tar tvf artifact.tar
```

## ETAPA 2: Preparar o Ambiente Alvo (Container-based)

```bash
# 1. Transferir artifact para o nó gateway do alvo
scp artifact.tar artifact.tar.sha256 usuario@gateway-alvo.empresa.org:~/

# 2. Verificar checksum
cd ~
sha256sum --check artifact.tar.sha256

# 3. Extrair
tar xf artifact.tar
cd artifact
sha256sum --check sha256sum.txt

# 4. Baixar installer containerizado (mesma versão do fonte)
# https://access.redhat.com/downloads → AAP 2.6 → Containerized bundle
```

## ETAPA 3: Instalar e Configurar o Ambiente Alvo

```bash
# Opção 1: Variáveis diretamente no inventory
egrep 'pg_database|_key' inventory
# Editar inventory com os valores do secrets.yml

# Opção 2: Arquivo de variáveis adicional (recomendado)
ansible-playbook -i inventory ansible.containerized_installer.install \
  -e @~/artifact/secrets.yml \
  -e "__hub_database_fields='{{ hub_db_fields_encryption_key }}'"

# Verificar versão do PostgreSQL (deve ser 15)
# Fazer backup do ambiente recém-instalado
ansible-playbook -i <path_to_inventory> ansible.containerized_installer.backup
```

## ETAPA 4: Importar o Conteúdo Migrado

```bash
# Parar serviços (exceto banco de dados)
# No nó do Controller:
systemctl --user stop automation-controller-task \
  automation-controller-web automation-controller-rsyslog
systemctl --user stop receptor

# No nó do Hub:
systemctl --user stop automation-hub-api automation-hub-content \
  automation-hub-worker

# No nó do Gateway:
systemctl --user stop automation-gateway

# Restaurar dumps
pg_restore -h <pg_hostname> -U <user> -d <database> \
  --clean --if-exists -Fc controller/controller.pgc

pg_restore -h <pg_hostname> -U <user> -d <database> \
  --clean --if-exists -Fc gateway/gateway.pgc

pg_restore -h <pg_hostname> -U <user> -d <database> \
  --clean --if-exists -Fc hub/hub.pgc

# Reiniciar serviços
systemctl --user start automation-controller-task \
  automation-controller-web automation-controller-rsyslog receptor \
  automation-hub-api automation-hub-content automation-hub-worker \
  automation-gateway
```

## ETAPA 5: Reconciliação e Validação

```bash
# Verificar integridade do Controller
awx-manage check_license
awx-manage print_settings | grep -i database

# Testar login na UI do alvo
# Verificar Job Templates, Inventories, Credentials
# Executar um job de teste

# Itens a recriar manualmente (fora do escopo da migração):
# - Instance Groups
# - EDA Controller config e rulebooks
# - Execution Environments customizados
# - Hub collections (via sync ou upload manual)
```
