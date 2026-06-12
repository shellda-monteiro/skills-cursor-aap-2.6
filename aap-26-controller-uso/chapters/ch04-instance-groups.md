# Ch04 — Instance Groups, Capacidade e Container Groups

## Hierarquia de Instance Groups

Quando um job é disparado, o Controller verifica os Instance Groups na seguinte ordem de prioridade:

```
Job Template  →  Inventory  →  Organization  →  default (grupo global)
```

O primeiro grupo que tiver capacidade disponível recebe o job. **O grupo definido no Job Template tem maior prioridade.**

## Criar e Usar Instance Groups

```
Controller UI → Automation Execution → Infrastructure → Instance Groups → Add
→ Nome: instance_group_producao
→ Minimum Forks: 0 (sem limite mínimo)
→ Maximum Forks: 50 (limitar paralelismo máximo neste grupo)

→ Instances → Add existing instances
→ Selecionar: exec-producao-1, exec-producao-2
```

### Associar ao Job Template

```
Job Template → Instance Groups → Selecionar: instance_group_producao
```

### Via API

```bash
# Criar instance group
curl -X POST -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"name": "instance_group_dmz", "max_forks": 20}' \
  https://gateway.empresa.org/api/controller/v2/instance_groups/

# Associar instância ao grupo
curl -X POST .../instance_groups/<id>/instances/ \
  -d '{"id": <instance_id>}'
```

## Regras de Nomenclatura de Instance Groups

```
✓ Nomear grupos com prefixo instance_group_ no inventory
✓ Membros de instance_group_* DEVEM estar em [automationcontroller] ou [execution_nodes]
✗ Não criar grupo chamado instance_group_default
✗ Não nomear uma instância igual ao nome de um grupo
✗ Não modificar o controlplane group (gera permission denied)
```

Exemplo no inventory:

```ini
[automationcontroller]
ctrl.exemplo.org node_type=control

[automationcontroller:vars]
peers=execution_nodes

[execution_nodes]
exec-local.exemplo.org   # também em instance_group_producao
exec-dmz.exemplo.org     # também em instance_group_dmz

[instance_group_producao]
exec-local.exemplo.org

[instance_group_dmz]
exec-dmz.exemplo.org
```

## Capacidade por Nó

A capacidade de um nó (forks disponíveis) é calculada automaticamente com base em RAM e CPU:

```
Capacidade = min(RAM / memória_por_fork, CPUs × fator)
```

Verificar via API:
```bash
curl -H "Authorization: Bearer <token>" \
  https://gateway.empresa.org/api/controller/v2/instances/
# Campos: capacity, consumed_capacity, percent_capacity_remaining
```

## Container Groups — Jobs em Pods OCP

Container Groups permitem rodar jobs como **pods efêmeros** no OpenShift — sem execution nodes dedicados.

```
Controller UI → Automation Execution → Infrastructure → Instance Groups → Add container group
→ Nome: cg-openshift-dev
→ Credential: OpenShift ou Kubernetes (kubeconfig/service account)
→ Pod Spec: (customizar namespace, CPU/memory requests/limits, nodeSelector, tolerations)
```

### Pod Spec customizado (exemplo)

```yaml
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: default
  automountServiceAccountToken: false
  containers:
    - image: >-
        registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest
      name: worker
      args:
        - ansible-runner
        - worker
        - '--private-data-dir=/runner'
      resources:
        requests:
          cpu: "500m"
          memory: "1Gi"
        limits:
          cpu: "2"
          memory: "4Gi"
  nodeSelector:
    role: automation
```

### Quando usar Container Groups

| Cenário | Usar Container Group? |
|---|---|
| Jobs esporádicos sem demanda constante | ✓ Sim — pods efêmeros, sem nó dedicado |
| Ambientes multi-tenant no OCP | ✓ Sim — isolamento por namespace |
| Jobs de alta frequência | ✗ Não — overhead de inicialização de pod |
| Execution nodes persistentes necessários | ✗ Não — usar Instance Groups normais |

## Capacidade e Políticas de Queue

```
Controller Settings → Jobs:
├── Maximum Scheduled Jobs: 10000 (limite de jobs no queue)
├── Job Events Maximum: 1000000 (máx eventos por job — limitar verbosity!)
└── Paginação de eventos: configurável por job
```

**Monitorar fila via API:**
```bash
# Jobs aguardando (pending)
GET /api/controller/v2/jobs/?status=pending

# Capacidade total do cluster
GET /api/controller/v2/instances/?order_by=hostname
```
