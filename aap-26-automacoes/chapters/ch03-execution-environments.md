# Ch03 — Execution Environments: Build e Deploy

## Por que usar Execution Environments?

- Elimina dependências conflitantes entre projetos (cada EE tem seu próprio ambiente)
- Garante reprodutibilidade: mesmo resultado em dev, homologação e produção
- Suporta múltiplos projetos com versões diferentes do mesmo pacote Python
- Substitui os Virtual Environments (vEnv) do AWX/Tower legado

## Estrutura do `execution-environment.yml` (v3)

```yaml
version: 3

# Imagem base — usar EE mínimo Red Hat como padrão
build_arg_defaults:
  ANSIBLE_GALAXY_CLI_COLLECTION_OPTS: "--pre"

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-26/ee-minimal-rhel9:latest

dependencies:
  # Collections Ansible
  galaxy:
    collections:
      - name: community.vmware          # Automação VMware
        version: ">=4.0.0"
      - name: redhat.satellite          # Red Hat Satellite
      - name: ansible.netcommon         # Automação de rede
      - name: ansible.windows           # Automação Windows
      - name: community.windows
      - name: ansible.platform          # CaC — gestão do AAP via API

  # Dependências Python
  python:
    - pyvmomi>=8.0.0                    # SDK VMware
    - ncclient>=0.6.15                  # NETCONF para rede
    - ntc-templates>=4.1.0              # TextFSM para parsing de rede
    - pywinrm[kerberos]>=0.4.3          # WinRM para Windows
    - requests>=2.31.0

  # Dependências de sistema (RPMs)
  system:
    - gcc [platform:rpm]
    - libffi-devel [platform:rpm]
    - python3-devel [platform:rpm]
    - krb5-workstation [platform:rpm]   # Kerberos para Windows

# Arquivos adicionais (ex: config de Galaxy)
additional_build_files:
  - src: ansible.cfg
    dest: src

# Passos customizados de build
additional_build_steps:
  prepend_galaxy:
    - ENV ANSIBLE_GALAXY_SERVER_URL=https://hub.exemplo.org/
    - ENV ANSIBLE_GALAXY_SERVER_AUTH_URL=https://hub.exemplo.org/auth/
  append_final:
    - RUN echo "Build completo"
```

## Instalação do ansible-builder

```bash
# Instalar a nível de usuário
pip3 install --user ansible-builder

# Verificar versão
ansible-builder --version
```

## Processo de Build

```bash
# 1. Criar o arquivo de definição
# (execution-environment.yml conforme acima)

# 2. Build da imagem
ansible-builder build \
  --file execution-environment.yml \
  --tag meu-ee:1.0.0 \
  --container-runtime podman \
  --verbosity 2

# 3. Testar localmente com ansible-navigator
ansible-navigator run meu_playbook.yml \
  --execution-environment-image meu-ee:1.0.0 \
  --mode stdout

# 4. Push para Private Automation Hub
podman login hub.exemplo.org
podman push meu-ee:1.0.0 hub.exemplo.org/organizacao/meu-ee:1.0.0

# 5. Criar EE no Controller
# Controller UI → Automation Execution Environments → Add
# Name: meu-ee, Image: hub.exemplo.org/organizacao/meu-ee:1.0.0
```

## Build em Ambiente Desconectado (Air-Gapped)

```yaml
# execution-environment.yml para air-gapped
version: 3

images:
  base_image:
    name: hub-interno.exemplo.org/rh/ee-minimal-rhel9:latest

dependencies:
  galaxy:
    collections:
      - name: community.vmware
        source: https://hub-interno.exemplo.org/api/galaxy/content/published/
```

```bash
# Configurar ansible.cfg para apontar para Hub interno
# ansible.cfg
[galaxy]
server_list = automation_hub

[galaxy_server.automation_hub]
url = https://hub-interno.exemplo.org/api/galaxy/content/published/
auth_url = https://hub-interno.exemplo.org/api/galaxy/v3/auth/token/
token = <token-do-hub>
```

## Imagens Base Disponíveis (Red Hat)

| Imagem | Conteúdo | Uso |
|---|---|---|
| `ee-minimal-rhel9` | ansible-core apenas | Base para builds customizados |
| `ee-supported-rhel9` | ansible-core + collections suportadas Red Hat | Uso geral |
| `ee-29-rhel9` | Compatibilidade com AAP 2.x anterior | Migração de legado |
| `de-minimal-rhel8` | Motor EDA apenas | Base para Decision Environments |
| `de-supported-rhel8` | Motor EDA + plugins suportados | Uso geral no EDA |

## Precedência de EEs no Controller

O Controller escolhe o EE na seguinte ordem de prioridade:
1. EE definido no **Job Template** (mais específico)
2. EE padrão da **Organization**
3. EE padrão **global** configurado no Controller

## Publicar EE no Private Hub via UI

1. Hub UI → Execution Environments → Add Remote
2. Informar URL: `registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9`
3. Credencial: `registry.redhat.io` com usuário RHN
4. Sincronizar → imagem disponível para todos os projetos da org
