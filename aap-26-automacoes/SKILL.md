---
name: aap-26-automacoes
description: >-
  Referência técnica AAP 2.6 para desenvolvimento de automações: Event-Driven Ansible (EDA),
  Rulebooks, Decision Environments, Configuration as Code (CaC), Execution Environments (EE),
  ansible-builder, e boas práticas de desenvolvimento de conteúdo Ansible.
  Use quando precisar configurar EDA, criar Rulebooks, webhooks, integração GLPI/ITSM,
  construir EEs customizados com ansible-builder, implementar CaC com ansible.platform collection,
  ou desenvolver roles e playbooks seguindo boas práticas AAP 2.6.
disable-model-invocation: true
---

# AAP 2.6 — Automações, EDA e Execution Environments

Referência extraída dos documentos oficiais: *Using automation decisions (EDA)*, *Configuration as Code*, *Creating and using execution environments*, e *Developing automation content* (AAP 2.6).

## Conceitos Centrais

### Event-Driven Ansible (EDA)
Automação reativa que escuta fontes de eventos e dispara ações automaticamente via Rulebooks.

```
Fonte de evento (webhook, Kafka, alert, ITSM)
        │
        ▼
  EDA Controller
  ├── Rulebook Activation (processo em execução)
  │   ├── Source Plugin   ← conecta à fonte
  │   ├── Condition       ← filtra eventos
  │   └── Action          ← run_job_template, run_playbook, etc.
  └── Decision Environment (imagem de contêiner)
```

### Execution Environments (EE)
Imagens OCI que encapsulam ansible-core + collections + dependências Python/sistema. Substituem os vEnvs do AAP 1.x.

```
EE = ansible-core + ansible-runner + collections + Python deps + system deps
```

### Configuration as Code (CaC)
Gerenciar configurações do AAP (Job Templates, Credenciais, Inventários) via YAML versionado em Git usando a `ansible.platform` collection.

## Índice de Capítulos

| Arquivo | Conteúdo |
|---|---|
| [chapters/ch01-eda-rulebooks.md](chapters/ch01-eda-rulebooks.md) | EDA, Rulebooks, fontes de eventos, condições, ações, webhooks |
| [chapters/ch02-eda-decision-environments.md](chapters/ch02-eda-decision-environments.md) | Decision Environments, configuração, performance, escalabilidade |
| [chapters/ch03-execution-environments.md](chapters/ch03-execution-environments.md) | EEs, ansible-builder, execution-environment.yml, build e push |
| [chapters/ch04-configuration-as-code.md](chapters/ch04-configuration-as-code.md) | CaC, ansible.platform collection, workflow Git+CI/CD |
| [chapters/ch05-desenvolvimento.md](chapters/ch05-desenvolvimento.md) | Boas práticas de roles, playbooks, coleções, ansible-navigator |
| [glossary.md](glossary.md) | Rulebook, Decision Environment, Source Plugin, EE, CaC, etc. |
| [patterns.md](patterns.md) | Padrões: EDA webhook, CaC GitOps, EE customizado vs padrão |
| [cheatsheet.md](cheatsheet.md) | Referência rápida: YAML Rulebook, execution-environment.yml, CaC |
