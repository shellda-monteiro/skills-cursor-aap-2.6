# Cheatsheet — AAP 2.6 Automações

## Rulebook — Estrutura Mínima

```yaml
---
- name: Nome do Rulebook
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
  rules:
    - name: Nome da regra
      condition: event.payload.campo == "valor"
      action:
        run_job_template:
          name: "Nome do Job Template"
          organization: "Nome da Org"
          job_args:
            extra_vars:
              chave: "{{ event.payload.campo }}"
```

## Source Plugins Principais

| Plugin | Pacote | Uso |
|---|---|---|
| `ansible.eda.webhook` | `ansible.eda` | HTTP POST genérico |
| `ansible.eda.kafka` | `ansible.eda` | Apache Kafka |
| `ansible.eda.alertmanager` | `ansible.eda` | Prometheus |
| `ansible.eda.aws_sqs_queue` | `ansible.eda` | AWS SQS |
| `ansible.eda.servicenow` | `servicenow.itsm` | ServiceNow |

## execution-environment.yml — Template

```yaml
version: 3
images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-26/ee-minimal-rhel9:latest
dependencies:
  galaxy:
    collections:
      - name: <nome_da_collection>
        version: ">=X.Y.Z"
  python:
    - <pacote_python>
  system:
    - <pacote_rpm> [platform:rpm]
```

## ansible-builder — Comandos

```bash
ansible-builder build --file ee.yml --tag meu-ee:1.0.0
ansible-builder create --file ee.yml     # só gera Containerfile
ansible-builder introspect --sanitize    # inspeciona dependências
podman push meu-ee:1.0.0 hub.org/ee:1.0.0
```

## ansible.platform — Módulos CaC

```yaml
# Conexão (repetir em cada task ou usar module_defaults)
controller_host: "https://gateway.org"
controller_username: "admin"
controller_password: "{{ aap_password }}"
validate_certs: true

# Objetos
ansible.platform.organization    # name, state
ansible.platform.team            # name, organization, state
ansible.platform.credential      # name, credential_type, inputs, organization
ansible.platform.inventory       # name, organization
ansible.platform.project         # name, organization, scm_url, scm_branch
ansible.platform.job_template    # name, playbook, project, inventory, credentials
ansible.platform.execution_environment  # name, image, credential
```

## ansible-lint — Profiles

| Profile | Nível | Quando |
|---|---|---|
| `min` | Mínimo | Migração de legado |
| `basic` | Básico | Desenvolvimento inicial |
| `moderate` | Moderado | Times em amadurecimento |
| `safety` | Segurança | Automações críticas |
| `shared` | Compartilhado | Collections compartilhadas |
| `production` | Produção | Padrão para CI/CD |

```bash
ansible-lint --profile production .
```

## Ações EDA — Referência Rápida

```yaml
# Disparar Job Template
action:
  run_job_template:
    name: "..."
    organization: "..."
    job_args:
      extra_vars: {}

# Disparar Workflow
action:
  run_workflow_template:
    name: "..."
    organization: "..."

# Logar (debug)
action:
  debug:
    msg: "{{ event | to_nice_json }}"

# Publicar evento interno (para outra regra)
action:
  post_event:
    event:
      campo: "{{ event.payload.campo }}"
```

## Checklist EDA — Integração Webhook

```
[ ] URL do EDA acessível pelo sistema externo (GLPI, ITSM)
[ ] Porta do source plugin liberada no firewall (ex: 5000 ou 443)
[ ] Credencial "Red Hat Ansible Automation Platform" criada no EDA
[ ] Decision Environment disponível no Private Hub
[ ] Projeto Git com Rulebooks sincronizado no EDA
[ ] Rulebook testado com action: debug primeiro
[ ] Payload mapeado e condições validadas
[ ] Rulebook Activation em status "Running"
```
