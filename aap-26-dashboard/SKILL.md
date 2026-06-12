---
name: aap-26-dashboard
description: >-
  Referência técnica do Automation Dashboard do AAP 2.6 — aplicação containerizada
  separada que coleta e consolida métricas de execução de jobs, eficiência e savings
  de múltiplos deployments AAP (até 3 instâncias da mesma versão). Cobre instalação
  em RHEL (Podman), integração via OAuth2 e clusters.yaml, sincronização de dados,
  métricas disponíveis (jobs/hora, taxa de falha, tempo médio, savings), troubleshooting
  de tokens e logs de container.
  Use quando precisar instalar o Automation Dashboard, conectá-lo a instâncias AAP,
  configurar sincronização periódica, resolver erros 401/404, ou entender as métricas
  de operação que o dashboard exibe.
disable-model-invocation: true
---

# AAP 2.6 — Automation Dashboard

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-instalacao](chapters/ch01-instalacao.md) | Pré-requisitos, inventory, instalação via ansible-playbook |
| [ch02-integracao](chapters/ch02-integracao.md) | clusters.yaml, OAuth2, sincronização, token rotation |
| [ch03-metricas](chapters/ch03-metricas.md) | Métricas disponíveis, visualizações, savings |
| [ch04-troubleshooting](chapters/ch04-troubleshooting.md) | Erros 401/404, logs de container, verificação de tokens |

## Uso Rápido

- **Instalar o Dashboard:** → ch01
- **Conectar a instância AAP existente:** → ch02 (clusters.yaml)
- **Sincronizar dados manualmente:** → ch02 (syncdata)
- **Erro 401 na sincronização:** → ch04
- **Quais métricas o dashboard exibe:** → ch03

## Diferença: Dashboard vs. Automation Analytics

| | Automation Dashboard | Automation Analytics |
|---|---|---|
| **Onde roda** | Servidor dedicado (Podman/RHEL) | console.redhat.com (SaaS Red Hat) |
| **Dados** | Jobs locais das instâncias conectadas | Telemetria enviada para RH HCC |
| **Instâncias** | Até 3 AAP da mesma versão | Ilimitado |
| **Air-gap** | Sim (totalmente local) | Não (requer internet) |
| **Savings calculator** | Sim (local) | Sim (cloud) |
