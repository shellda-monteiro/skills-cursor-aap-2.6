# Ch01 — EDA: Rulebooks, Fontes e Ações

## Visão Geral do EDA Controller

O EDA Controller processa eventos de múltiplas fontes e executa ações automaticamente, sem intervenção humana.

**Componentes principais:**
- **Rulebook Activation**: processo em execução que monitora fontes de eventos
- **Decision Environment**: imagem de contêiner com o motor de regras (análogo ao EE para o Controller)
- **Source Plugin**: conector para a fonte de eventos (webhook, Kafka, alertas, etc.)
- **Credentials**: autenticação para projetos Git e Decision Environments

## Estrutura de um Rulebook

```yaml
# meu_rulebook.yml
---
- name: Processar chamados do GLPI
  hosts: all
  sources:
    - ansible.eda.webhook:               # source plugin
        host: 0.0.0.0
        port: 5000
        token: "{{ webhook_token }}"     # autenticação do webhook

  rules:
    - name: Criar VM ao abrir chamado de provisionamento
      condition: event.meta.headers.X-GLPI-Type == "provisioning"
      action:
        run_job_template:
          name: "Provisionar VM vCenter"
          organization: "TI-Infraestrutura"
          job_args:
            extra_vars:
              vm_nome: "{{ event.payload.hostname }}"
              vm_cpu: "{{ event.payload.cpu }}"
              vm_ram: "{{ event.payload.ram }}"
```

## Sources Plugins Disponíveis (de-minimal — AAP 2.6)

Os plugins nativos do `de-minimal` são listados abaixo. O namespace foi migrado de `ansible.eda.*` para `eda.builtin.*` — usar os nomes novos em novos rulebooks.

### Plugins nativos do de-minimal (suportados)

| Plugin (nome atual) | Tipo | Nome depreciado |
|---|---|---|
| `eda.builtin.dashes_to_underscores` | Event filter | `ansible.eda.dashes_to_underscores` |
| `eda.builtin.event_splitter` | Event filter | — |
| `eda.builtin.generic` | Event source | `ansible.eda.generic` |
| `eda.builtin.insert_hosts_to_meta` | Event filter | `ansible.eda.insert_hosts_to_meta` |
| `eda.builtin.insert_meta_info` | Event filter | — |
| `eda.builtin.json_filter` | Event filter | `ansible.eda.json_filter` |
| `eda.builtin.normalize_keys` | Event filter | `ansible.eda.normalize_keys` |
| `eda.builtin.pg_listener` | Event source | `ansible.eda.pg_listener` |
| `eda.builtin.range` | Event source | `ansible.eda.range` |
| `eda.builtin.webhook` | Event source | `ansible.eda.webhook` |

### Plugins no de-supported (requerem `de-supported` ou coleções dedicadas)

| Event source (de-supported) | Nome depreciado (de-minimal) |
|---|---|
| `amazon.aws.aws_cloudtrail` | `ansible.eda.aws_cloudtrail` |
| `amazon.aws.aws_sqs_queue` | `ansible.eda.aws_sqs_queue` |
| `azure.azcollection.azure_service_bus` | `ansible.eda.azure_service_bus` |

### Plugins a serem removidos em versão futura (evitar novos usos)
- `ansible.eda.file`
- `ansible.eda.file_watch`
- `ansible.eda.journald`
- `ansible.eda.tick`
- `ansible.eda.url_check`

## Condições (Conditions)

As condições são expressões em Python que filtram eventos recebidos:

```yaml
# Verificar campo no payload
condition: event.payload.status == "open"

# Verificar header HTTP
condition: event.meta.headers["Content-Type"] == "application/json"

# Condição composta
condition: event.payload.priority == "high" and event.payload.category == "infra"

# Verificar se campo existe
condition: event.payload.hostname is defined

# Filtro por origem
condition: event.meta.source.name == "glpi-webhook"
```

## Ações Disponíveis

```yaml
# Disparar Job Template no Controller
action:
  run_job_template:
    name: "Nome do Job Template"
    organization: "Minha Org"
    job_args:
      extra_vars:
        variavel: "{{ event.payload.campo }}"

# Executar playbook diretamente (EDA managed)
action:
  run_playbook:
    name: "playbooks/resposta.yml"

# Disparar Workflow Template
action:
  run_workflow_template:
    name: "Workflow de Remediação"
    organization: "Ops"

# Apenas logar evento (debug)
action:
  debug:
    msg: "Evento recebido: {{ event }}"

# Postvar — criar variável para usar na próxima regra
action:
  post_event:
    event:
      category: "{{ event.payload.category }}"
```

## Configuração do Webhook (Firewall)

Para receber webhooks externos, o EDA Controller precisa ser acessível:

```ini
# Inventory: habilitar porta do webhook
[automationeda:vars]
eda_main_url=https://eda.exemplo.org
# Porta padrão: 443 (via Platform Gateway)
# Porta do source plugin: configurada no Rulebook (ex: 5000)
```

**Regra de firewall necessária:**
```
Origem (GLPI/ITSM) → EDA Controller : TCP 443 (ou porta do source plugin)
```

## Projetos no EDA

Os Rulebooks ficam em repositórios Git. Criar um **Projeto** no EDA:

1. EDA Controller UI → Projects → Add
2. Informar URL do repositório Git (HTTPS ou SSH)
3. Informar credencial (Username/Password ou SSH Key)
4. O EDA sincroniza automaticamente

## Rulebook Activations

Após criar o Projeto, ativar o Rulebook:

1. EDA UI → Rulebook Activations → Add
2. Selecionar: Projeto, Rulebook, Decision Environment
3. Definir variáveis extras (se necessário)
4. Ativar — o EDA inicia o processo de escuta

**Status esperados:**
- `Running` → escutando eventos normalmente
- `Failed` → verificar logs de ativação (credenciais, Decision Environment, conectividade)

## Performance e Escalabilidade

```yaml
# Inventory: horizontal scaling do EDA
[automationeda]
eda1.exemplo.org
eda2.exemplo.org   # adicionar mais nós para escalar

# Configurar número de workers por ativação
eda_workers: 4     # padrão: 2, máx: 8 (recomendado)
```

**Regra de dimensionamento EDA:**
- 1 worker por Rulebook Activation concorrente
- Aumentar `eda_workers` se houver muitas ativações simultâneas
- Cada nó EDA pode rodar múltiplas ativações em paralelo
