---
name: aap-26-self-service
description: >-
  Referência técnica do Self-Service Automation Portal do AAP 2.6 — interface
  web simplificada para usuários de negócio executarem automações Ansible sem
  conhecimento de playbooks. Construído sobre Red Hat Developer Hub (RHDH/Backstage),
  instalado via Helm chart no OpenShift. Cobre instalação (Helm + OCP), pré-requisitos
  (OAuth2 no AAP, secrets OCP), sincronização de Job Templates via AAP provider,
  templates customizados (Custom Self-Service Templates), RBAC por template,
  gerenciamento de coleções e EEs no portal, e troubleshooting de SSL e autenticação.
  Use quando precisar instalar o portal, configurar sincronização de templates com AAP,
  criar formulários customizados para usuários não-técnicos, definir RBAC por template,
  ou resolver erros de login/SSL.
disable-model-invocation: true
---

# AAP 2.6 — Self-Service Automation Portal

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-arquitetura](chapters/ch01-arquitetura.md) | O que é, arquitetura (RHDH/Backstage), limitações, pré-requisitos |
| [ch02-instalacao](chapters/ch02-instalacao.md) | Pré-configuração OAuth2, secrets OCP, Helm values, instalação |
| [ch03-sincronizacao](chapters/ch03-sincronizacao.md) | Sincronização de Job Templates, filtros por label/survey, Helm config |
| [ch04-uso](chapters/ch04-uso.md) | Login, templates auto-gerados, templates customizados, RBAC |

## Uso Rápido

- **O que é o portal:** → ch01
- **Instalar no OpenShift via Helm:** → ch02
- **Sincronizar apenas templates com label específica:** → ch03
- **Criar formulário customizado para usuário não-técnico:** → ch04
- **Erro de SSL no login:** → ch04 (troubleshooting SSL)

## Conceito Central

```
AAP Job Template
    ↓ sincronização automática
Self-Service Portal (RHDH)
    ↓ interface ponto-e-clique
Usuário de negócio
    → executa o Job Template sem ver YAML
```
