# Ch01 — Job Templates: Configuração e Padrões

## Campos de um Job Template

| Campo | Descrição | Prompt on Launch? |
|---|---|---|
| **Name** | Nome do template | Não |
| **Inventory** | Inventory a ser usado | Sim |
| **Project** | Projeto Git com os playbooks | Não |
| **Playbook** | Arquivo .yml a executar | Não |
| **Credentials** | Credenciais para execução | Sim |
| **Variables** | Extra vars (YAML ou JSON) | Sim |
| **Limit** | Filtro de hosts do inventory | Sim |
| **Labels** | Etiquetas para filtrar por domínio | Sim |
| **Job Tags / Skip Tags** | `--tags` / `--skip-tags` | Sim |
| **Forks** | Paralelismo (padrão: 5) | Sim |
| **Job Slice Count** | Distribuir inventory em N fatias | Não |
| **Execution Environment** | Imagem de container para execução | Não |
| **Instance Groups** | Restringe execução a grupos específicos | Não |
| **Concurrent Jobs** | Permitir execuções simultâneas | Não |
| **Verbosity** | Nível de detalhe do output | Sim |

> **Prompt on Launch:** permite sobrescrever o valor no momento do launch (via UI ou API), útil para templates reutilizáveis entre ambientes.

## Surveys (Formulários de Input)

Surveys tornam um Job Template parametrizável via formulário — sem expor o playbook:

```
Template → Survey → Adicionar questão:
  - Tipo: Text, Textarea, Password, Integer, Float, Multiple Choice (single/multiple)
  - Nome da variável: destino_host
  - Label para o usuário: "Hostname do servidor alvo"
  - Obrigatório: sim/não
  - Valor padrão: db.empresa.org
```

**Uso no playbook:**
```yaml
- name: Usar variável do survey
  debug:
    msg: "Alvo: {{ destino_host }}"
```

## Job Slicing — Paralelismo Horizontal

**Quando usar:** playbooks que rodam independentemente em cada host (sem coordenação cruzada). Ex: patching, coleta de configuração, deploy de agente.

```
Job Template → Job Slice Count: 4
→ Inventory com 400 hosts
→ AAP divide em 4 fatias de 100 hosts e dispara 4 jobs em paralelo
→ Resultado: um Workflow Job com 4 sub-jobs
```

**Regras:**
- `slice_count ≤ nó de execution disponíveis` é o recomendado
- Não usar para playbooks que orquestram **entre** hosts (ex: cluster rolling restart)
- Limit passado ao sliced job aplica-se a TODAS as fatias — se o limit deixar uma fatia vazia, ela **falha**

## Schedules

Qualquer Job Template ou Workflow pode ter schedules:

```
Template → Schedules → Add
→ Nome: "Patching semanal"
→ Start: 2025-02-01 22:00
→ Repeat: Weekly, domingo
→ Timezone: America/Sao_Paulo
```

Schedule com extra vars:
```json
{"ambiente": "producao", "reboot_allowed": false}
```

## Webhooks — Disparar Template via SCM

```
Job Template → Enable Webhook → Webhook Service: GitHub ou GitLab
→ Webhook URL: https://gateway.empresa.org/api/controller/v2/job_templates/5/github/
→ Webhook Key: copiar e configurar no repositório GitHub → Settings → Webhooks
```

Push para o repositório → Controller recebe evento → lança o Job Template automaticamente.

## Boas Práticas para Consultoria

```
✓ Usar Surveys para parâmetros que variam por cliente/ambiente
✓ "Prompt on Launch" em Inventory e Credentials — mesmo template, vários ambientes
✓ Labels para organizar templates por projeto/cliente
✓ Job Slicing para coletas de configuração em 100+ hosts
✓ Schedules com janela de manutenção definida (timezone correto!)
✓ Concurrent Jobs: desabilitado por padrão — habilitar apenas se necessário
✓ Execution Environment específico por template (garantir reprodutibilidade)
```
