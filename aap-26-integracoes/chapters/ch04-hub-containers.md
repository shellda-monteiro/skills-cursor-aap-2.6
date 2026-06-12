# Ch04 — Private Automation Hub: Container Registry

## Visão Geral

O Private Automation Hub funciona como **registry interno de container** para a organização. Armazena Execution Environments (EEs) aprovados, controlando quais imagens os times podem usar para executar automação.

**Fluxo padrão:**
```
registry.redhat.io (fonte) → pull local → tag → push para Private Hub → Controller usa o Hub
```

## Permissões de Times para Containers

| Permissão | Descrição |
|---|---|
| `Create new containers` | Criar novos repositórios de container |
| `Change container namespace permissions` | Alterar permissões no repositório |
| `Change container` | Alterar informações do container |
| `Change execution environment tags` | Modificar tags do EE |
| `Push to existing container` | Fazer push de EE para container existente |

Configurar: Hub UI → Access Management → Teams → Permissões

## Workflow: Publicar EE no Private Hub

### 1. Pull da imagem do Red Hat Registry

```bash
# Login no registry.redhat.io
podman login registry.redhat.io

# Pull da imagem desejada
podman pull registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest

# Verificar
podman images | grep ee-supported
```

### 2. Tag para o Hub privado

```bash
# Formato: <hub_fqdn>/<namespace>/<nome_imagem>:<tag>
podman tag \
  registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest \
  hub.empresa.org/minhaempresa/ee-supported:2.6
```

### 3. Login e Push para o Private Hub

```bash
# Login no Hub privado
podman login -u=admin -p=<senha> hub.empresa.org

# Push
podman push hub.empresa.org/minhaempresa/ee-supported:2.6

# Verificar via API
curl -k -H "Authorization: Bearer <token>" \
  https://hub.empresa.org/api/galaxy/_ui/v1/execution-environments/
```

### 4. Configurar EE no Controller

```
Controller UI → Automation Execution → Infrastructure → Execution Environments → Add
→ Image: hub.empresa.org/minhaempresa/ee-supported:2.6
→ Pull: Always (ou Only if not present)
→ Credential: credencial de registry do Hub (se Hub não for público)
```

## Build de EE Customizado e Push para Hub

```bash
# 1. Buildar
ansible-builder build \
  --file execution-environment.yml \
  --tag hub.empresa.org/minhaempresa/ee-customizado:1.0 \
  --container-runtime podman

# 2. Push direto para o Hub
podman push hub.empresa.org/minhaempresa/ee-customizado:1.0
```

## Assinatura de Imagens (Signing Service)

O Hub suporta assinatura de imagens com GPG para garantir integridade:

```bash
# Script de assinatura (pulp_container SigningService)
#!/bin/bash
MANIFEST_PATH=$1
FINGERPRINT="$PULP_SIGNING_KEY_FINGERPRINT"
IMAGE_REFERENCE="$REFERENCE"
SIGNATURE_PATH="$SIG_PATH"

# Assinar com skopeo
skopeo standalone-sign \
  $MANIFEST_PATH \
  $IMAGE_REFERENCE \
  $FINGERPRINT \
  --output $SIGNATURE_PATH

# (Opcional) Proteger com passphrase
# --passphrase-file /path/to/key_password.txt
```

Configurar o Signing Service no Hub e associar ao namespace para assinatura automática durante push.

## Conectar Automation Hub ao Controller

Para usar o Hub como fonte de EEs e Collections no Controller:

```
Controller UI → Platform Settings → System → Automation Hub URL
→ URL: https://hub.empresa.org/api/galaxy/
→ Authentication token: <token do Hub>
```

Ou via inventory (instalação):

```ini
[all:vars]
automationhub_main_url=https://hub.empresa.org
```
