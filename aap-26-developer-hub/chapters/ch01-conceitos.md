# Ch01 — Conceitos: RHDH e Ansible Plug-ins

## O que é Red Hat Developer Hub (RHDH)

Plataforma IDP (Internal Developer Platform) open-source baseada em **Backstage** (Spotify) que as organizações usam para criar portais internos de desenvolvedor: catálogo de software, templates de projeto, documentação técnica, integração com ferramentas (CI/CD, monitoramento, etc.).

## O que os Ansible Plug-ins Adicionam

Ao instalar os Ansible Plug-ins em um RHDH existente, a interface ganha:

| Funcionalidade | Descrição |
|---|---|
| **Home page Ansible** | Navegação e painel personalizado para Ansible |
| **Learning Paths** | Trilhas de aprendizado curadas (hosted em developers.redhat.com) |
| **Labs** | Laboratórios práticos para desenvolvimento de conteúdo |
| **Software Templates** | Templates para criar projetos de playbook e collection no Git |
| **Discover Collections** | Link para Private Automation Hub ou console.redhat.com |
| **OpenShift Dev Spaces** | IDE web sob demanda para desenvolvimento de automação |
| **Operate (Controller link)** | Link direto ao Automation Controller para criar e executar jobs |

## Workflow Completo (Learn → Operate)

```
RHDH → Ansible → Overview
  │
  ├── Learn     → Learning Paths + Labs (developers.redhat.com)
  │
  ├── Discover  → Private Automation Hub (coleções e EEs internos)
  │               ou console.redhat.com (hub público Red Hat)
  │
  ├── Create    → Software Template → cria repositório Git com estrutura
  │               padrão de projeto Ansible (playbook ou collection)
  │
  ├── Develop   → OpenShift Dev Spaces → IDE web com:
  │               - ansible-navigator pré-configurado
  │               - EE corporativo configurado
  │               - Ansible extension no VS Code
  │               - Lightspeed (se configurado)
  │
  └── Operate   → Automation Controller → configurar Project + Job Template
                  → executar jobs com o conteúdo desenvolvido
```

## Diferença: Ansible Plug-ins vs Self-Service Portal

| | Ansible Plug-ins (RHDH) | Self-Service Portal |
|---|---|---|
| **Público** | Desenvolvedores/automadores | Usuários de negócio |
| **Propósito** | Criar novo conteúdo Ansible | Executar automações existentes |
| **RHDH** | Adicionar ao RHDH existente | RHDH dedicado (instalado junto) |
| **Subscription** | AAP + RHDH existente | AAP (RHDH incluído) |

> As duas soluções podem coexistir na mesma organização servindo públicos diferentes.
