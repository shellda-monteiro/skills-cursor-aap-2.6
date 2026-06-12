---
name: aap-26-developer-hub
description: >-
  Referência técnica dos Ansible Plug-ins para Red Hat Developer Hub (RHDH) do
  AAP 2.6. O RHDH é um portal IDP (Internal Developer Platform) baseado em
  Backstage onde os plug-ins Ansible adicionam: home page personalizada para Ansible,
  learning paths curados, software templates para criar projetos de playbook/collection
  no Git (GitHub/GitLab), integração com OpenShift Dev Spaces para IDEs web, e link
  direto ao Automation Controller para operar jobs. Cobre instalação via Helm chart
  ou Operator no OCP, os dois métodos de entrega de plug-ins (OCI recomendado vs
  HTTP registry), configuração obrigatória e opcional, e o workflow completo de uso
  (Learn → Discover → Create → Develop → Operate).
  Use quando precisar instalar os Ansible plug-ins no RHDH existente, criar software
  templates para padronizar novos projetos Ansible, ou entender o workflow integrado
  Developer Hub → Dev Spaces → Controller.
disable-model-invocation: true
---

# AAP 2.6 — Ansible Plug-ins para Red Hat Developer Hub

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-conceitos](chapters/ch01-conceitos.md) | O que é RHDH, o que os plug-ins adicionam, workflow completo |
| [ch02-instalacao](chapters/ch02-instalacao.md) | Pré-requisitos, métodos de entrega (OCI/HTTP), Helm + Operator |
| [ch03-uso](chapters/ch03-uso.md) | Dashboard, Learn, Create project, Develop (Dev Spaces), Operate (Controller) |

## Uso Rápido

- **O que os plug-ins Ansible adicionam ao RHDH:** → ch01
- **Instalar plug-ins no RHDH existente:** → ch02
- **Criar template para padronizar novos projetos Ansible:** → ch03
- **Diferença RHDH Ansible Plug-ins vs Self-Service Portal:** → ch01

## Comparação com Self-Service Portal

| | Ansible Plug-ins (RHDH) | Self-Service Portal |
|---|---|---|
| **Foco** | **Desenvolvedores** Ansible (criar projetos, aprender) | **Usuários de negócio** (executar automações existentes) |
| **Base** | RHDH existente na organização | RHDH instalado pelo portal |
| **Instalação** | Plug-ins adicionados ao RHDH | Aplicação separada |
| **Público** | Automadores, desenvolvedores | Consumidores de automação |
