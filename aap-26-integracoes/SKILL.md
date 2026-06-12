---
name: aap-26-integracoes
description: >-
  Referência técnica do Red Hat Ansible Automation Platform 2.6 cobrindo
  integrações com HashiCorp (Terraform + Vault), automação de segurança,
  gerenciamento de coleções e imagens de contêiner no Private Automation Hub,
  configuração avançada do Automation Controller (secret management, security
  best practices, awx-manage), e backup/restore para ambientes Operator (OCP).
  Use quando precisar de integração Terraform/Vault, sincronizar coleções no Hub,
  publicar Execution Environments, configurar credential types customizados para
  HashiCorp, fazer backup do Controller no OCP, ou usar awx-manage para
  operações de linha de comando.
disable-model-invocation: true
---

# AAP 2.6 — Integrações, Hub e Controller Avançado

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-terraform](chapters/ch01-terraform.md) | Integração Ansible+Terraform: workflows, TF Actions, EDA+Terraform |
| [ch02-vault](chapters/ch02-vault.md) | HashiCorp Vault: autenticação, credential types, migração de community.hashi_vault |
| [ch03-hub-collections](chapters/ch03-hub-collections.md) | Gerenciar coleções no Hub: certified, validated, Galaxy, sync, aprovação |
| [ch04-hub-containers](chapters/ch04-hub-containers.md) | Private Hub como registry de EEs: push, assinatura, signing service |
| [ch05-controller-avancado](chapters/ch05-controller-avancado.md) | Secret management, awx-manage, security best practices, backup OCP |

## Uso Rápido

- **Integrar Terraform com AAP:** → ch01
- **Usar Vault como credential provider:** → ch02
- **Sincronizar collections do Red Hat Hub:** → ch03
- **Publicar imagem EE no Private Hub:** → ch04
- **Operações de linha de comando no Controller:** → ch05
- **Backup/restore no OCP Operator:** → ch05
