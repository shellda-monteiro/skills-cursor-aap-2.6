---
name: aap-26-controller-uso
description: >-
  Referência técnica do Automation Controller do AAP 2.6 cobrindo Job Templates,
  Workflow Job Templates (Workflow Visualizer), Job Slicing, Schedules, Inventories
  (estáticos, dinâmicos e plugins), Instance Groups e Container Groups (hierarquia
  JT > Inventory > Org), capacidade e forks, credentials e custom credential types,
  notificações e webhooks, e API REST (autenticação, filtragem, paginação).
  Use quando precisar de configurações de Job Templates, como montar workflows
  complexos com aprovação, qual plugin de inventory dinâmico usar (VMware, AWS,
  ServiceNow), como segregar execução via Instance Groups, como integrar CI/CD
  via webhooks ou API, ou como calcular capacidade de jobs por nó.
disable-model-invocation: true
---

# AAP 2.6 — Uso do Automation Controller

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-job-templates](chapters/ch01-job-templates.md) | Job Templates: campos, Prompt on Launch, surveys, slicing, schedules |
| [ch02-workflows](chapters/ch02-workflows.md) | Workflow Templates, Visualizer, nós de convergência e aprovação |
| [ch03-inventories](chapters/ch03-inventories.md) | Inventories estáticos, dinâmicos, plugins (VMware, AWS, ServiceNow, etc.) |
| [ch04-instance-groups](chapters/ch04-instance-groups.md) | Instance Groups, hierarquia de prioridade, Container Groups (OCP) |
| [ch05-api-webhooks](chapters/ch05-api-webhooks.md) | API REST (auth, filter, sort, pagination), webhooks SCM (GitHub/GitLab) |

## Uso Rápido

- **Criar Job Template com variáveis dinâmicas:** → ch01 (Prompt on Launch + Surveys)
- **Pipeline com aprovação humana:** → ch02 (Workflow + nó de aprovação)
- **Inventory dinâmico do VMware/AWS:** → ch03
- **Isolar execução por ambiente/rede:** → ch04 (Instance Groups)
- **Integrar GitLab CI → AAP:** → ch05 (webhooks)
- **Chamar API do Controller de script:** → ch05
