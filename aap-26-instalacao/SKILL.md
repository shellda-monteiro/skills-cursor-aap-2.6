---
name: aap-26-instalacao
description: >-
  Referência técnica completa do Red Hat Ansible Automation Platform 2.6 cobrindo
  arquitetura, instalação containerizada (Podman), instalação via Operator (OpenShift),
  topologias testadas, requisitos de sistema, rede/portas, Redis, PostgreSQL e certificados TLS.
  Use quando precisar de requisitos de instalação AAP, topologias growth vs enterprise,
  portas de firewall, variáveis de inventory, pré-requisitos de storage/IOPS, StorageClass
  OpenShift, ou decisões de arquitetura entre containerizado e Operator.
disable-model-invocation: true
---

# AAP 2.6 — Arquitetura e Instalação

Referência extraída dos documentos oficiais Red Hat: *Planning your installation*, *Containerized installation*, *Installing on OpenShift Container Platform*, e *Tested deployment models* (AAP 2.6).

## Modelos de Deploy (decisão central)

| Modo | Infra | Gerenciado por | Topologias |
|---|---|---|---|
| **Containerizado** | VMs / bare metal (RHEL 9.4+) | Cliente (Podman) | Growth · Enterprise |
| **Operator** | Red Hat OpenShift | Cliente (OCP Operator) | Growth · Enterprise |
| RPM | VMs (RHEL 9 apenas) | Cliente | Deprecated em 2.5 → removido em 2.7 |

**Regra de decisão:** Use **Containerizado** quando a infra é RHEL puro sem OCP. Use **Operator** quando o cliente já opera OpenShift — o ciclo de vida é gerenciado pelo Operator Hub, mas exige StorageClass RWX e namespace dedicado.

## Componentes da Plataforma

```
Platform Gateway  ──┐
Automation Controller─┤──→ PostgreSQL (15/16/17)
Automation Hub     ──┤──→ Redis (centralized/standalone)
EDA Controller     ──┘
        │
        └──→ Automation Mesh (Receptor 27199)
                   ├── Execution Nodes
                   └── Hop Nodes
```

- **Platform Gateway**: ponto único de entrada, autenticação (LDAP/SAML), RBAC, JWT
- **Automation Controller**: orquestração de jobs, inventários, credenciais
- **Automation Hub**: repositório de EEs e Collections (Pulp backend + PostgreSQL)
- **EDA Controller**: event-driven automation via Rulebooks; usa Redis para event queues
- **Redis**: cache/fila centralizado para Gateway e EDA; Controller e Hub têm instâncias próprias
- **Execution Environments (EE)**: imagens de contêiner com ansible-core + collections + deps Python

## Topologias e Requisitos Mínimos

### Growth (all-in-one) — Containerizado
- **1 VM**: 16 GB RAM · 4 vCPUs · 60 GB disco · **3000 IOPS**
- `redis_mode=standalone`
- Todos os componentes no mesmo host (gateway, controller, hub, eda, database)
- 32 GB RAM obrigatório se `hub_seed_collections=true` (seeding ~45+ min)

### Enterprise — Containerizado
- **Múltiplos hosts** com separação de componentes
- Mínimo por componente: 8 GB RAM · 2 vCPUs
- Redis em modo **cluster** (6 VMs: 3 primários + 3 réplicas)
- Controller **não deve** compartilhar VM com Hub ou banco de dados

### Operator Growth / Enterprise (OpenShift)
- Requer OpenShift 4.x compatível
- Automation Hub: **StorageClass ReadWriteMany (RWX)** obrigatória ou object storage (S3/Azure Blob)
- Deploy em **namespace dedicado** (não usar `default`)
- Usuário com `cluster-admin` para instalar o Operator

## Pré-Requisitos Críticos (Containerizado)

1. **RHEL 9.4+** ou RHEL 10 (x86_64, AArch64, s390x, ppc64le)
2. **Usuário não-root com sudo** — dados em `$HOME/aap/` e `$HOME/.local/share/containers/`
3. **ansible-core**: 2.14 no RHEL 9 · 2.16 no RHEL 10 (instalar via `dnf install ansible-core`); a collection `ansible.containerized_installer` vem empacotada no installer
4. **NTP configurado** em todos os nós (falhas de JWT e certificados sem NTP)
5. **DNS/FQDN** resolvível para todos os nós — sem underscores (`_`) nos hostnames
6. **Podman NÃO suporta NFS** para home directory — usar disco local para `$HOME/.local/share/containers/`
7. **registry_username/registry_password** válidos para `registry.redhat.io` (online) ou bundle installer (offline)

## Índice de Capítulos

| Arquivo | Conteúdo |
|---|---|
| [chapters/ch01-componentes.md](chapters/ch01-componentes.md) | Componentes, Redis, PostgreSQL, Automation Mesh |
| [chapters/ch02-topologias.md](chapters/ch02-topologias.md) | Topologias testadas: requisitos, exemplos de inventory |
| [chapters/ch03-containerizado.md](chapters/ch03-containerizado.md) | Instalação Containerizada: `ansible.containerized_installer.install`, inventory, TLS, DB externo |
| [chapters/ch04-openshift-operator.md](chapters/ch04-openshift-operator.md) | Instalação via Operator: OCP, StorageClass, custom resources |
| [chapters/ch05-rede-portas.md](chapters/ch05-rede-portas.md) | Portas de rede, protocolos, regras de firewall, quay.io/CDN |
| [glossary.md](glossary.md) | Termos técnicos: EE, EDA, CaC, Receptor, Mesh, Hub, etc. |
| [patterns.md](patterns.md) | Quando usar growth vs enterprise, containerizado vs Operator, Redis standalone vs cluster |
| [cheatsheet.md](cheatsheet.md) | Tabela rápida de portas, IOPS, versões, variáveis críticas do inventory |
