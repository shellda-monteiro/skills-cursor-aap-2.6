# Ch03 — Automation Mesh no OpenShift (Operator)

## Diferença: Control Plane no OpenShift

No modelo Operator (OpenShift), o **control plane** não usa nós hybrid ou control — é formado por **container groups** (pods) rodando no cluster Kubernetes. O execution plane usa VMs RHEL externas conectadas via Receptor.

| Componente | OpenShift Operator |
|---|---|
| Control plane | Pods no cluster OCP (controller-task, controller-web, etc.) |
| Execution plane | VMs RHEL externas com Receptor (via mesh ingress) |
| Hop nodes | VMs RHEL com Receptor, node_type=hop |

## Pré-requisitos para Execution Nodes Externos

Cada VM que será execution node precisa:
1. RHEL com IP estático ou DNS resolvível
2. Subscrição AAP (10 RHEL licenses incluídas na subscrição AAP)
3. Habilitação do repositório AAP:

```bash
# RHEL 9 x86_64
subscription-manager repos \
  --enable ansible-automation-platform-2.6-for-rhel-9-x86_64-rpms

# RHEL 8 x86_64 (legado)
subscription-manager repos \
  --enable ansible-automation-platform-2.5-for-rhel-8-x86_64-rpms
```

## Adicionar Execution Node via UI (sem re-instalação)

No OpenShift, é possível adicionar execution nodes **sem rodar o installer novamente**:

1. Platform Gateway UI → Automation Execution → Infrastructure → **Instances**
2. Click **Add** → configurar hostname, porta e tipo de nó
3. O Operator gera os certificados e configura o Receptor automaticamente
4. Associar o novo Instance ao Instance Group desejado

> **Vantagem OCP vs VM/Container:** Scaling dinâmico sem downtime — não requer re-run do installer.

## Instance Groups no OpenShift

```bash
# UI: Automation Execution → Infrastructure → Instance Groups
# Criar grupo → adicionar instances → associar a Job Templates/Organizations
```

Casos de uso comuns:
- `instance_group_dmz` — execution nodes na DMZ para ativos de rede
- `instance_group_aws` — execution nodes em cloud AWS (mais próximos dos targets)
- `instance_group_prod` — nodes dedicados apenas para produção

## Scaling OCP vs VM-based — Comparação

| Operação | OpenShift Operator | VM-based / Containerizado |
|---|---|---|
| Scale horizontal | Ajustar replicas no CR — sem downtime | Re-run installer — requer downtime |
| Scale vertical | Ajustar requests/limits no Operator | Re-run installer após resize da VM |
| Upgrade | Rolling update automático pelo Operator | Re-run installer — serviços reiniciam |
| Agregação de logs | Stack de logging do OCP (nativo) | Soluções externas (ELK, Splunk) |
| Monitoramento | Prometheus + Grafana nativos no OCP | Ferramentas externas |

## Mesh Ingress (OCP → Execution Nodes Externos)

Para que execution nodes externos (VMs) se conectem ao OCP, usa-se uma **Route** do OpenShift como ponto de entrada:

```
VM (execution node) → HTTPS 443 → OCP Route (mesh ingress) → Receptor no OCP
```

**Regra de firewall necessária:**
```
Execution nodes → OCP Route URL : TCP 443 (outbound)
```

Configurar no inventory:
```ini
# Receptor listener nos execution nodes externos
receptor_listener_port=27199
```

## CPU Throttling no OCP — IMPORTANTE

No OCP, definir CPU limit sem CPU request equivalente causa **throttling**, mesmo com CPU disponível:

```yaml
# ERRADO — causa throttling
resources:
  limits:
    cpu: "4"
  # sem requests

# CORRETO — requests = limits
resources:
  requests:
    cpu: "4"
  limits:
    cpu: "4"
```

Monitorar throttling via métrica: `container_cpu_cfs_throttled_seconds_total`
