# Ch02 — Topologias Testadas e Dimensionamento

## Container Growth Topology (All-in-One)

**Caso de uso:** Organizações iniciando com AAP, footprint pequeno, sem redundância.

### Requisitos da VM única
| Recurso | Mínimo | Observação |
|---|---|---|
| RAM | 16 GB | 32 GB se `hub_seed_collections=true` |
| vCPUs | 4 | — |
| Disco total | 60 GB | — |
| Disco instalação | 15 GB | Se partição dedicada |
| /var/tmp (online) | 1 GB | — |
| /var/tmp (bundle) | 3 GB | — |
| /tmp (bundle) | 10 GB | — |
| **IOPS de disco** | **3000** | Crítico para PostgreSQL |

### Componentes no mesmo host
- automationgateway, automationcontroller, automationhub, automationeda, database

### Inventory exemplo (growth, online)
```ini
[automationgateway]
aap.example.org

[automationcontroller]
aap.example.org

[automationhub]
aap.example.org

[automationeda]
aap.example.org

[database]
aap.example.org

[all:vars]
ansible_connection=local
postgresql_admin_username=postgres
postgresql_admin_password=<senha>
registry_username=<RHN user>
registry_password=<RHN password>
redis_mode=standalone
gateway_admin_password=<senha>
gateway_pg_host=aap.example.org
gateway_pg_password=<senha>
controller_admin_password=<senha>
controller_pg_host=aap.example.org
controller_pg_password=<senha>
hub_admin_password=<senha>
hub_pg_host=aap.example.org
hub_pg_password=<senha>
eda_admin_password=<senha>
eda_pg_host=aap.example.org
eda_pg_password=<senha>
```

## Container Enterprise Topology

**Caso de uso:** Produção, redundância, alta demanda de jobs concorrentes.

### Separação de componentes (VMs distintas)
- **Platform Gateway** nodes (1+): 8 GB RAM, 2 vCPUs
- **Automation Controller** nodes (1+): 16 GB RAM, 4 vCPUs
- **Automation Hub** nodes (1+): 8 GB RAM, 2 vCPUs
- **EDA Controller** nodes (enterprise = dedicado): 8 GB RAM, 2 vCPUs
- **Database**: 16 GB RAM, 4 vCPUs — **não compartilhar com Controller**
- **Redis cluster**: 6 nós (pode colocar nos hosts de componentes, exceto Controller, Execution nodes e DB)

### Regras de colocation enterprise
- Controller e Hub: **NUNCA** no mesmo nó
- Controller e Database: **NUNCA** no mesmo nó
- Hub e Controller no mesmo banco: **só se databases diferentes** (`awx` ≠ `automationhub`)
- Execution nodes: sem Redis, sem banco

## Operator Growth Topology (OpenShift)

| Recurso OpenShift | Requisito |
|---|---|
| Namespace | Dedicado (não `default`) |
| StorageClass (Automation Hub) | **ReadWriteMany (RWX)** ou S3/Azure Blob |
| Permissão do usuário | `cluster-admin` para instalar o Operator |
| PostgreSQL | Gerenciado pelo Operator (PVC) ou externo |

### Custom Resource mínimo (growth)
```yaml
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatform
metadata:
  name: aap
  namespace: aap
spec:
  gateway:
    replicas: 1
  controller:
    replicas: 1
  hub:
    replicas: 1
    storage_type: file         # ou s3, azure
  eda:
    replicas: 1
```

## Operator Enterprise Topology (OpenShift)

- Gateway, Controller, Hub, EDA com replicas > 1
- Redis via Operator com HA
- PostgreSQL externo recomendado para produção
- PVCs com StorageClass RWX ou object storage para Hub

## Requisitos de SO (Containerizado)

| Item | Valor |
|---|---|
| SO suportado | RHEL 9.4+ ou RHEL 10+ |
| Arquiteturas | x86_64, AArch64, s390x, ppc64le |
| ansible-core (RHEL 9) | 2.14 (`dnf install ansible-core`) |
| ansible-core (RHEL 10) | 2.16 |
| Browser | Firefox ou Chrome suportados |
| IP | IPv4, IPv6, dual-stack |

### Setup obrigatório do host RHEL
```bash
# 1. Registrar e habilitar repositórios
sudo subscription-manager register
sudo subscription-manager attach --auto
sudo subscription-manager repos --enable ansible-automation-platform-2.6-for-rhel-9-x86_64-rpms

# 2. Instalar ansible-core
sudo dnf install -y ansible-core

# 3. Configurar usuário não-root com sudo (NOPASSWD)
sudo visudo -f /etc/sudoers.d/aap-user
# Adicionar: aapuser ALL=(ALL) NOPASSWD: ALL

# 4. Configurar NTP (crítico!)
sudo timedatectl set-ntp true

# 5. Verificar DNS — FQDN deve resolver em todos os nós
hostnamectl set-hostname aap.example.org

# 6. Utilitários opcionais para troubleshooting
sudo dnf install -y wget git-core rsync vim
```
