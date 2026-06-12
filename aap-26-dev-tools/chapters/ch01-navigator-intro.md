# Ch01 — Automation Content Navigator: Introdução e Comandos

## O que é o ansible-navigator

Ferramenta CLI/TUI (Text-based UI) para desenvolver e executar conteúdo Ansible dentro de **Execution Environments (EEs)**. Parte do pacote **Ansible Development Tools (ADT)**.

> É o substituto moderno do `ansible-playbook` direto — garante que o playbook rode no mesmo ambiente que rodará no Controller.

## Dois Modos de Operação

| Modo | Como usar | Quando usar |
|---|---|---|
| **stdout** | `ansible-navigator run playbook.yml -m stdout` | Saída direta, familiar como ansible-playbook, scripts |
| **TUI (text-based UI)** | `ansible-navigator run playbook.yml` | Desenvolvimento interativo, inspeção, debug |

> Não é possível trocar de modo durante a execução.

## Tabela de Comandos

| Comando navigator | Equivalente Ansible | Descrição |
|---|---|---|
| `ansible-navigator collections` | `ansible-galaxy collection` | Explorar coleções disponíveis |
| `ansible-navigator config` | `ansible-config` | Ver configuração atual do Ansible |
| `ansible-navigator doc` | `ansible-doc` | Documentação de módulos e plugins |
| `ansible-navigator images` | — | Explorar EEs disponíveis |
| `ansible-navigator inventory` | `ansible-inventory` | Explorar um inventory |
| `ansible-navigator replay` | — | Revisar artefato de run anterior |
| `ansible-navigator run` | `ansible-playbook` | Executar um playbook |
| `ansible-navigator welcome` | — | Tela de boas-vindas (TUI) |

## Atalhos no Modo TUI

| Tecla/Comando | Ação |
|---|---|
| `:run` | Executar playbook |
| `:collections` | Listar coleções |
| `:images` | Listar EEs |
| `:doc <módulo>` | Ver documentação do módulo |
| `:replay <arquivo>` | Replay de artefato |
| `:<número>` | Ir para linha/item específico |
| `Esc` | Voltar à tela anterior |
| `:help` | Ajuda |

## Instalação

```bash
# Instalar Ansible Development Tools (inclui navigator)
pip3 install --user ansible-dev-tools

# Ou via dnf (RHEL)
sudo dnf install ansible-dev-tools

# Verificar instalação
ansible-navigator --version
```
