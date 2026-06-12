---
name: aap-26-analytics-migracao
description: >-
  Referência técnica do AAP 2.6 cobrindo dois temas essenciais para consultoria:
  (1) Migração entre deployment types — caminhos suportados (RPM→Container,
  RPM→OpenShift, Container→OpenShift, qualquer tipo→Managed AAP), processo passo a
  passo de exportação do ambiente fonte, criação do migration artifact, importação
  no ambiente alvo e reconciliação pós-migração; (2) Automation Analytics —
  Automation Calculator (ROI e fórmulas de savings), Automation Savings Planner,
  Reports, Job Explorer e Data Dictionary.
  Use quando precisar planejar migração de AAP RPM para containerizado ou OpenShift,
  calcular ROI de automação para cliente, apresentar relatórios de savings ou
  analisar execuções via Job Explorer.
disable-model-invocation: true
---

# AAP 2.6 — Migração e Analytics

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-migracao-caminhos](chapters/ch01-migracao-caminhos.md) | Caminhos suportados, escopo, regra da mesma versão, pré-requisitos |
| [ch02-migracao-processo](chapters/ch02-migracao-processo.md) | Export do fonte, migration artifact, import no alvo, reconciliação |
| [ch03-automation-calculator](chapters/ch03-automation-calculator.md) | Fórmulas de ROI, Top Templates, variáveis de custo |
| [ch04-automation-analytics](chapters/ch04-automation-analytics.md) | Savings Planner, Reports, Job Explorer, Data Dictionary |

## Uso Rápido

- **Cliente tem AAP RPM, quer migrar para containerizado:** → ch01 + ch02
- **Apresentar ROI de automação para gestor:** → ch03 (Automation Calculator)
- **Planejar e documentar savings por iniciativa:** → ch04 (Savings Planner)
- **Investigar jobs com falha em todos os clusters:** → ch04 (Job Explorer)
