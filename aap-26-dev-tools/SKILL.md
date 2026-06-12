---
name: aap-26-dev-tools
description: >-
  Referência técnica de ferramentas de desenvolvimento Ansible do AAP 2.6 cobrindo
  dois componentes: (1) Automation Content Navigator (ansible-navigator) — ferramenta
  CLI/TUI para desenvolver playbooks dentro de Execution Environments, inspecionar
  coleções e EEs, executar e replay de runs, configuração via ansible-navigator.yml;
  (2) Red Hat Ansible Lightspeed with IBM watsonx Code Assistant — serviço de IA
  generativa integrado ao VS Code que gera recomendações de tasks e playbooks a partir
  de prompts em linguagem natural, disponível como cloud service ou on-premise.
  Use quando precisar configurar o ansible-navigator para desenvolvimento local,
  rodar playbooks dentro de um EE específico, inspecionar inventories/coleções via TUI,
  ativar o Lightspeed no VS Code, ou entender como configurar o modelo de IA para
  a organização.
disable-model-invocation: true
---

# AAP 2.6 — Ferramentas de Desenvolvimento (Navigator + Lightspeed)

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-navigator-intro](chapters/ch01-navigator-intro.md) | Comandos, modos (stdout/TUI), instalação |
| [ch02-navigator-uso](chapters/ch02-navigator-uso.md) | Executar playbooks em EEs, explorar coleções, inventories, configuração |
| [ch03-lightspeed-intro](chapters/ch03-lightspeed-intro.md) | O que é, benefícios, cloud vs on-premise, requisitos |
| [ch04-lightspeed-uso](chapters/ch04-lightspeed-uso.md) | Ativar no VS Code, gerar tasks/playbooks, administração, Code Bot |

## Uso Rápido

- **Rodar playbook dentro de um EE específico:** → ch02 (ansible-navigator run)
- **Listar coleções disponíveis no EE:** → ch02 (:collections no TUI)
- **Configurar ansible-navigator.yml do projeto:** → ch02
- **Ativar Lightspeed no VS Code:** → ch04
- **Entender trial vs. subscrição paga:** → ch03
