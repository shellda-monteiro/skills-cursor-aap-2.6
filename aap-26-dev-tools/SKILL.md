---
name: aap-26-dev-tools
description: >-
  Referência técnica de ferramentas de desenvolvimento Ansible do AAP 2.6 cobrindo
  três componentes do Lightspeed e desenvolvimento: (1) Automation Content Navigator
  (ansible-navigator) — ferramenta CLI/TUI para desenvolver playbooks dentro de
  Execution Environments, inspecionar coleções e EEs, executar e replay de runs,
  configuração via ansible-navigator.yml; (2) Red Hat Ansible Lightspeed Coding
  Assistant — serviço de IA generativa integrado ao VS Code que gera recomendações
  de tasks e playbooks a partir de prompts em linguagem natural, disponível como cloud
  service ou on-premise; (3) Automation Intelligent Assistant — chatbot de IA embutido
  na UI do AAP que responde perguntas sobre o ambiente em tempo real usando LLMs
  (RHEL AI, OCP AI, OpenAI, Azure) com integração ao MCP server (aap_gateway_url +
  aap_controller_url). MCP GA no OCP, Tech Preview no containerizado.
  Use quando precisar configurar o ansible-navigator, ativar o Lightspeed no VS Code,
  implantar o chatbot inteligente no AAP, configurar o secret chatbot-configuration-secret,
  habilitar o MCP server para o assistente buscar dados reais do Controller, ou trocar
  o modelo LLM do chatbot.
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
| [ch05-intelligent-assistant](chapters/ch05-intelligent-assistant.md) | Automation Intelligent Assistant, MCP server, deploy no OCP, configuração LLM |

## Uso Rápido

- **Rodar playbook dentro de um EE específico:** → ch02 (ansible-navigator run)
- **Listar coleções disponíveis no EE:** → ch02 (:collections no TUI)
- **Configurar ansible-navigator.yml do projeto:** → ch02
- **Ativar Lightspeed no VS Code:** → ch04
- **Entender trial vs. subscrição paga:** → ch03
- **Implantar chatbot de IA no AAP (OCP):** → ch05
- **Habilitar MCP para o chatbot buscar dados do Controller:** → ch05
- **Configurar chatbot-configuration-secret:** → ch05
