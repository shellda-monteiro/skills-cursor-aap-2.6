# Cheatsheet — Mesh, Performance e Upgrade

## Node Types Rápido

```
node_type=hybrid     # control + execution (padrão RPM)
node_type=control    # só controle, sem executar playbooks
node_type=execution  # só executa playbooks
node_type=hop        # relay de tráfego — não executa
```

```
# Containerizado: usar receptor_type= (não node_type=)
receptor_type=control | execution | hop
receptor_peers=host1.org,host2.org   # não usar nomes de grupos
```

## Topologias em Snapshot

| Padrão | Inventory key |
|---|---|
| HA control plane | 3 hybrid nodes em [automationcontroller] |
| Control separado | node_type=control + peers=execution_nodes |
| DMZ/Remoto | hop node + instance_group_remote peers=hop |
| Multi-hop | hop1 → hop2 → execution-remote |
| Outbound-only | execution peers=automationcontroller |

## Peering (RPM)

```ini
[automationcontroller:vars]
peers=execution_nodes   # peer todos os control com todos os execution
```

## Performance Quick Wins

```
# Reduzir carga da UI
UI_LIVE_UPDATES_ENABLED=false

# Usar Token auth na API (mais rápido que Basic)
Authorization: Bearer <token>

# EDA alto volume: desabilitar audit eventos
Rulebook Activation → Skip audit events: ON

# Manter job events por poucos dias
Maximum days before deleting stdout: 30
```

## Upgrade Paths Rápido

```
RPM 2.4 RHEL 9 → RPM 2.6       : upgrade direto
RPM 2.5 RHEL 9 → RPM 2.6       : upgrade direto
RPM 2.5 RHEL 9 → Container 2.6 : upgrade → migrate
RPM 2.4 RHEL 8 → Container 2.6 : backup/restore RHEL 9 → upgrade 2.6 → migrate
Qualquer → RHEL 10              : backup/restore no RHEL 10 (não in-place)
```

## RHEL Suportado

```
RPM 2.6         : RHEL 9 only
Container 2.6   : RHEL 9 ou RHEL 10
OCP 2.6         : conforme OCP lifecycle
```
