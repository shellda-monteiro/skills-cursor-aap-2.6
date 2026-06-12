# Ch02 — Automation Mesh: Design Patterns e Topologias

## Padrão 1: Múltiplos Hybrid Nodes (HA Control Plane)

**Caso de uso:** Redundância no control plane. Todos os nós hybrid fazem tudo.

```ini
[automationcontroller]
ctrl1.exemplo.org
ctrl2.exemplo.org
ctrl3.exemplo.org

[automationcontroller:vars]
node_type=hybrid   # padrão — pode omitir
```

Nós do control plane são automaticamente peered uns com os outros. Adicionar novo nó ao grupo → auto-peer.

---

## Padrão 2: Control Plane Separado + Execution Node (Recomendado)

**Caso de uso:** Separar responsabilidades. Control nodes = apenas gerenciamento. Execution nodes = playbooks.

```ini
[automationcontroller]
ctrl1.exemplo.org
ctrl2.exemplo.org

[automationcontroller:vars]
node_type=control
peers=execution_nodes   # todos os control nodes peeiam com todos os execution nodes

[execution_nodes]
exec1.exemplo.org
exec2.exemplo.org

[execution_nodes:vars]
node_type=execution
```

**Configuração mínima resiliente:**
- 2 control nodes (HA automático entre si)
- 2 execution nodes (alcançáveis por qualquer control node)

---

## Padrão 3: Segregado com Hop Node (DMZ / Remoto)

**Caso de uso:** Execution nodes em rede remota, DMZ ou local de baixa conectividade. O hop node serve como relay.

```ini
[automationcontroller]
ctrl1.exemplo.org
ctrl2.exemplo.org

[automationcontroller:vars]
node_type=control
peers=instance_group_local   # peeiar apenas com execution nodes locais

[execution_nodes]
exec1.exemplo.org    # local
exec2.exemplo.org    # local
hop1.exemplo.org node_type=hop   # hop node (não executa automação)
exec3.exemplo.org    # remoto — atrás do hop

[instance_group_local]
exec1.exemplo.org
exec2.exemplo.org

[hop]
hop1.exemplo.org

[hop:vars]
peers=automationcontroller   # hop conecta-se ao controller

[instance_group_remote]
exec3.exemplo.org

[instance_group_remote:vars]
peers=hop   # execution remoto conecta-se ao hop
```

**Fluxo de tráfego:** Controller → Hop → Execution (remoto)

---

## Padrão 4: Multi-hop

**Caso de uso:** Múltiplos saltos de rede para alcançar execution nodes isolados.

```ini
[automationcontroller]
ctrl.exemplo.org

[automationcontroller:vars]
node_type=control
peers=hop1

[hop_1]
hop1.exemplo.org

[hop_1:vars]
node_type=hop
peers=automationcontroller   # hop1 conecta ao controller

[hop_2]
hop2.exemplo.org

[hop_2:vars]
node_type=hop
peers=hop_1   # hop2 conecta ao hop1

[execution_nodes_remote]
exec-remote.exemplo.org

[execution_nodes_remote:vars]
peers=hop_2   # execution conecta ao hop2
```

---

## Padrão 5: Outbound-Only Connections (Firewall restritivo)

**Caso de uso:** Execution nodes não podem receber conexões de entrada — apenas saída para o Controller.

```ini
# A conexão é estabelecida *de* execution nodes *para* automationcontroller
[execution_nodes]
exec-outbound.exemplo.org

[execution_nodes:vars]
peers=automationcontroller   # execution conecta-se ao controller (não o contrário)
```

> **Quando usar:** Ambientes com firewall que bloqueia inbound para execution nodes (ex: nodes em cloud sem porta 27199 aberta para o controller).

---

## Configuração Containerizado — Diferenças Chave

Em instalações containerizadas, substituir:

| RPM | Containerizado |
|---|---|
| `node_type=` | `receptor_type=` |
| `peers=` | `receptor_peers=` (lista de hostnames, não grupos) |
| `peers=execution_nodes` | `receptor_peers=exec1.org,exec2.org` |

```ini
# Exemplo containerizado — segregado com hop
[automationcontroller]
ctrl.exemplo.org receptor_type=control receptor_peers=exec1.exemplo.org,hop1.exemplo.org

[execution_nodes]
exec1.exemplo.org receptor_type=execution
hop1.exemplo.org receptor_type=hop receptor_peers=ctrl.exemplo.org
exec-remote.exemplo.org receptor_type=execution receptor_peers=hop1.exemplo.org
```

## Instance Groups — Associar Jobs a Execution Nodes Específicos

```bash
# Via UI: Automation Execution → Infrastructure → Instance Groups → Create
# Adicionar execution nodes ao grupo
# Associar Job Templates ao Instance Group específico
```

```yaml
# Via CaC (ansible.platform collection)
- ansible.platform.instance_group:
    name: "dmz-network"
    instances:
      - exec-dmz-1.exemplo.org
      - exec-dmz-2.exemplo.org

- ansible.platform.job_template:
    name: "Backup Rede"
    instance_groups:
      - "dmz-network"
```
