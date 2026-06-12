# Ch02 — Workflow Job Templates e Visualizer

## O que são Workflows

Workflows encadeiam Job Templates, Workflow Templates, syncs de projeto e syncs de inventory em um grafo de execução. Cada nó pode ter três estados de saída: **success**, **failure** e **always**.

```
Workflow Job Template
    ├── Nó A (Job Template: validar pré-requisitos)
    │   ├── [success] → Nó B (Deploy aplicação)
    │   │   ├── [success] → Nó D (Smoke test)
    │   │   └── [failure] → Nó E (Notificar falha)
    │   └── [failure] → Nó F (Notificar e abortar)
    └── [always] → Nó G (Gravar relatório)
```

## Tipos de Nós no Workflow

| Tipo de nó | O que faz |
|---|---|
| **Job Template** | Executa um playbook |
| **Workflow Template** | Sub-workflow (recursivo) |
| **Project Sync** | Atualiza o repositório Git do projeto |
| **Inventory Sync** | Sincroniza uma fonte de inventory |
| **Approval** | Pausa o workflow — aguarda aprovação humana |

## Nó de Convergência

Múltiplos nós podem convergir em um único nó downstream — o nó convergente só dispara quando **todos** os predecessores terminam:

```
Job A ─┐
       ├─→ [convergência] → Notificar resultado
Job B ─┘
```

Configuração: arrastar dois nós para o mesmo nó no Workflow Visualizer.

## Nó de Aprovação Humana

```
Workflow Visualizer → Adicionar nó → Tipo: Approval
→ Timeout: 30 minutos (após timeout: continua ou falha, configurável)
→ Notificação: enviar e-mail/Slack para aprovador
```

Quando o workflow chega ao nó de aprovação, fica pausado até alguém clicar em Approve ou Deny na UI.

**Caso de uso:** deploy em produção após validação em staging — requer aprovação do time de QA.

## Passagem de Variáveis entre Nós (Artifacts)

```yaml
# Playbook nó A: publicar resultado para nós downstream
- name: Registrar versão deployada
  set_stats:
    data:
      versao_deployada: "{{ app_version }}"
      url_aplicacao: "https://app.empresa.org"
    per_host: false
```

```yaml
# Playbook nó B (downstream): usar a variável
- name: Testar aplicação deployada
  uri:
    url: "{{ url_aplicacao }}/health"
    status_code: 200
```

> `set_stats` com `per_host: false` → variável disponível para **todos** os nós downstream. Em convergência, se múltiplos nós definirem a mesma chave, a mesclagem é indefinida — usar chaves únicas.

## Workflow Aninhado (Sub-workflow)

Workflows podem ser usados como nós dentro de outros workflows:

```
Workflow Principal
    ├── Sub-workflow: Provisionar VM (VMware + Satellite)
    │   ├── Job: Criar VM no vCenter
    │   ├── Job: Registrar no Satellite
    │   └── Job: Configurar repositórios
    └── Job: Deploy da aplicação
```

## Configurar Webhook no Workflow Template

```
Workflow Template → Enable Webhook → GitHub/GitLab
→ Disparar o workflow quando há push no repositório de infrastructure
```

## Campos do Workflow Template com "Prompt on Launch"

| Campo | Pode ser sobrescrito no launch? |
|---|---|
| Inventory | Sim |
| Limit | Sim (se marcado; não herda de job templates filhos automaticamente) |
| Labels | Sim |
| Job Tags / Skip Tags | Sim |
| Source control branch | Sim (aplicado a todos os nós que aceitem branch) |
| Extra variables | Sim (surveys ou extra_vars na launch) |

> **Atenção:** Limit definido no Workflow Template **não** é passado automaticamente para os Job Templates filhos — cada filho precisa ter `Prompt on Launch → Limit` marcado.

## Boas Práticas de Workflows para Consultoria

```
✓ Separar playbooks em etapas pequenas (validar → executar → testar → notificar)
✓ Usar nó de Aprovação em pipelines de produção
✓ Usar convergência para paralelizar etapas independentes (ex: deploy multi-região)
✓ Usar set_stats para passar contexto entre etapas (versão, URL, status)
✓ Sub-workflows para encapsular processos reutilizáveis (ex: provisionar VM)
✓ Notificação em falha → Slack/e-mail com link direto para o job falho
✓ Inventory no nível do Workflow (não por job) quando todos usam o mesmo inventory
```
