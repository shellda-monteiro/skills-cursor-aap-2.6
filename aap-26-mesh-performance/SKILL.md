---
name: aap-26-mesh-performance
description: >-
  Referência técnica do Red Hat Ansible Automation Platform 2.6 cobrindo
  Automation Mesh (VM e OpenShift/Operator), tipos de nós (hybrid, control,
  execution, hop), design patterns de topologia, peering, TLS do Receptor,
  performance tuning (tipos de workload, scaling, forks, event-driven),
  performance no OpenShift (pod specs, control plane adjustments, dedicated
  nodes), e suporte matrix de upgrade (2.4→2.6, 2.5→2.6, RPM→Container→OCP).
  Use quando precisar de inventories de mesh, topologia com hop nodes, regras de
  peering, escalabilidade do Controller, tipos de jobs (sliced/workflow/bulk),
  caminhos de upgrade suportados ou migração entre deployment types.
disable-model-invocation: true
---

# AAP 2.6 — Automation Mesh, Performance e Upgrade

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-mesh-conceitos](chapters/ch01-mesh-conceitos.md) | Tipos de nós, planes, peers, node_type vs receptor_type |
| [ch02-mesh-topologias](chapters/ch02-mesh-topologias.md) | Design patterns: hybrid, segregado, hop nodes, multi-hop, outbound-only |
| [ch03-mesh-openshift](chapters/ch03-mesh-openshift.md) | Mesh no OpenShift Operator, instance groups, VMs como execution nodes |
| [ch04-performance-tuning](chapters/ch04-performance-tuning.md) | Tipos de workload, scaling, forks, EDA activations, API performance |
| [ch05-upgrade-support-matrix](chapters/ch05-upgrade-support-matrix.md) | Support matrix completo: 2.4/2.5→2.6, RPM→Container→OCP |

## Uso Rápido

- **Topologia multi-hop (DMZ):** → ch02
- **Adicionar execution node remoto:** → ch01 (node_type) + ch02 (segregated)
- **Scalability: quantos forks/workers?** → ch04
- **Upgrade de 2.4 RHEL 8 → Container 2.6:** → ch05
- **Execution nodes no OpenShift:** → ch03
