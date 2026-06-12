# Ch03 — Gerenciamento de Collections no Private Automation Hub

## Tipos de Conteúdo no Hub

| Tipo | Origem | Suporte | Acesso |
|---|---|---|---|
| **Certified Collections** | Red Hat + parceiros | Suporte Red Hat + parceiros | Requer subscrição AAP |
| **Validated Collections** | Red Hat | Expert-led, testadas | Requer subscrição AAP |
| **Galaxy Collections** | Comunidade Ansible | Sem suporte | Livre (ansible-galaxy) |

## Configurar Sincronização com console.redhat.com

### 1. Obter API Token do console.redhat.com

```
1. Acessar https://console.redhat.com/ansible/automation-hub/token/
2. Automation Hub → Connect to Hub
3. Offline token → Load Token → Copiar
```

> **ATENÇÃO:** Token offline expira após 30 dias de inatividade.

### 2. Configurar Remote no Private Hub

```
Private Hub UI → Automation Content → Remotes
→ rh-certified: informar token e URL do console.redhat.com
→ Salvar → Sync
```

### 3. Habilitar coleções na instalação (inventory)

```ini
[all:vars]
# Fazer seed de collections durante a instalação
hub_seed_collections=true                    # habilitar seeding (padrão: true)
automationhub_collection_seed_repository=certified  # ou: validated, ou omitir para ambos
```

> **AVISO:** `hub_seed_collections=true` na topologia growth requer **32 GB RAM** e pode demorar 45+ minutos.

## Criar Namespace e Publicar Collection Interna

### 1. Criar time com permissões de namespace

```
Hub UI → Access Management → Teams → Create
→ Permissões: Add namespace, Upload to namespace
```

### 2. Criar namespace

```
Hub UI → Automation Content → Namespaces → Create
→ Nome: minhaempresa
→ Associar ao time criado
```

### 3. Fazer upload de collection via CLI

```bash
# Buildar a collection
cd ~/meu-projeto/collections/ansible_collections/minhaempresa/minha_collection/
ansible-galaxy collection build

# Configurar o cliente para o Hub privado
cat > ~/.ansible/galaxy_token << EOF
---
token: <token-do-hub-privado>
EOF

# Upload
ansible-galaxy collection publish \
  minhaempresa-minha_collection-1.0.0.tar.gz \
  --server https://hub.empresa.org \
  --api-key <token>
```

### 4. Aprovar a collection (admin)

```
Hub UI → Automation Content → Collections → [aguardando aprovação]
→ Approve → Publicar no namespace
```

## Instalar Collections em Execution Environments

```yaml
# requirements.yml no projeto do EE
collections:
  - name: minhaempresa.minha_collection
    version: ">=1.0.0"
  - name: ansible.netcommon
    version: ">=5.0.0"
  - name: community.vmware
    version: ">=3.0.0"
```

```yaml
# execution-environment.yml
version: 3
images:
  base_image:
    name: 'registry.redhat.io/ansible-automation-platform-26/ee-minimal-rhel9:latest'
dependencies:
  galaxy: requirements.yml
additional_build_env:
  prepend_galaxy: |
    [galaxy]
    server_list = automation_hub, galaxy
    [galaxy_server.automation_hub]
    url=https://hub.empresa.org/api/galaxy/
    auth_url=https://hub.empresa.org/api/galaxy/v3/auth/token/
    token=<token>
    [galaxy_server.galaxy]
    url=https://galaxy.ansible.com/
```

## Sincronizar via Installer (Ansible Automation Platform Migration)

```bash
# Re-sincronizar coleções após upgrade
cd /path/to/aap-installer/
ansible-playbook -i inventory ansible.containerized_installer.install --tags configure_hub
```

## Verificar Assinatura de Collection

```bash
# requirements.yml com assinatura
collections:
  - name: minha.colecao
    version: 1.0.0
    signatures:
      - https://hub.empresa.org/detached_signature.asc

# Verificar
ansible-galaxy collection verify \
  -r requirements.yml \
  --keyring ~/.ansible/pubring.kbx
```
