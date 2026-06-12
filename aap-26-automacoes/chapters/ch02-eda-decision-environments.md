# Ch02 — Decision Environments e Credenciais EDA

## Decision Environments

Decision Environments são imagens de contêiner que contêm o motor de execução dos Rulebooks do EDA. São análogos aos Execution Environments do Automation Controller.

### DE padrão Red Hat
```
registry.redhat.io/ansible-automation-platform-26/de-minimal-rhel8:latest
registry.redhat.io/ansible-automation-platform-26/de-supported-rhel8:latest
```

### Construir DE customizado

```yaml
# decision-environment.yml
version: 3

base_image:
  name: registry.redhat.io/ansible-automation-platform-26/de-minimal-rhel8:latest

dependencies:
  galaxy:
    collections:
      - name: ansible.eda
      - name: community.general
  python:
    - requests>=2.28.0
    - boto3>=1.26.0   # se usar source plugins AWS
```

```bash
# Instalar ansible-builder
pip3 install --user ansible-builder

# Construir a imagem
ansible-builder build \
  --file decision-environment.yml \
  --tag meu-de:latest \
  --container-runtime podman

# Push para Private Hub
podman push meu-de:latest <hub-fqdn>/organizacao/meu-de:latest
```

## Credenciais no EDA Controller

O EDA usa credenciais para:
1. Sincronizar Projetos Git (Rulebooks)
2. Autenticar com o Automation Controller (para `run_job_template`)
3. Autenticar para baixar Decision Environments do Hub

### Tipos de credenciais no EDA

| Tipo | Uso |
|---|---|
| `Source Control` | Acesso a repositório Git de Rulebooks |
| `Container Registry` | Pull de Decision Environments de registry privado |
| `Red Hat Ansible Automation Platform` | Conexão EDA → Controller para disparar jobs |
| `Vault` | Acesso a HashiCorp Vault para segredos |

### Configurar conexão EDA → Controller

```yaml
# Credential Type: Red Hat Ansible Automation Platform
# Campos obrigatórios:
URL: https://controller.exemplo.org
Username: admin
Password: <senha>
# OU usar OAuth Token em vez de username/password
```

## Variáveis de Ambiente nos Rulebooks

```yaml
- name: Rulebook com segredos injetados
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: processar evento
      condition: event.payload is defined
      action:
        run_job_template:
          name: "Meu Template"
          organization: "Ops"
          job_args:
            extra_vars:
              # Variáveis de ambiente são injetadas automaticamente
              # via Rulebook Activation → Extra Vars
              api_token: "{{ lookup('env', 'API_TOKEN') }}"
```

## Troubleshooting de Ativações EDA

```bash
# Via CLI do Controller (se tiver acesso ao host)
# Ver logs do processo de ativação
journalctl -u automation-eda-activations -f

# Via UI: EDA Controller → Rulebook Activations → [nome] → History

# Problemas comuns:
# 1. Decision Environment não encontrado → verificar credencial de registry
# 2. Falha ao conectar ao Controller → verificar credencial AAP
# 3. Webhook não recebido → verificar firewall e porta do source plugin
# 4. Condição nunca satisfeita → usar action: debug para inspecionar evento
```

## Payload de Webhook — Inspecionar Estrutura

Ao integrar com novos sistemas (como GLPI), use `debug` primeiro para ver o payload:

```yaml
- name: Debug GLPI webhook
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: Logar todos os eventos
      condition: event is defined
      action:
        debug:
          msg: |
            Headers: {{ event.meta.headers }}
            Payload: {{ event.payload | to_nice_json }}
```

Após identificar a estrutura real do payload, criar as condições e ações corretas.
