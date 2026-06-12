# Ch01 — Componentes, Redis e Automation Mesh

## Componentes e Dependências

### Platform Gateway
- Serviço de autenticação e autorização central (único ponto de entrada)
- Suporta LDAP, SAML, e SSO institucional
- Armazena no Redis: Settings, Session Information, JSON Web Tokens
- Porta de exposição: 80/443 (HTTP/HTTPS)

### Automation Controller
- Framework de orquestração: Job Templates, Inventários, Credenciais, Workflows
- Redis próprio: job launching
- Conecta ao PostgreSQL na porta 5432
- Não colocar no mesmo nó que o Automation Hub ou o banco de dados

### Automation Hub (Private)
- Repositório de EEs (container registry) e Collections (Pulp)
- Requer PostgreSQL com extensão **hstore** habilitada
- Para instalação via Operator: **obrigatório StorageClass RWX** ou object storage (S3/Azure Blob)
- Não usar FQDN com underscore (`_`) — Skopeo rejeita
- Redis próprio para gestão de conteúdo

### EDA Controller (Event-Driven Ansible)
- Interface para Rulebooks e automação reativa
- Consome eventos de múltiplas fontes (webhooks, Kafka, alertas)
- **Usa Redis centralizado** compartilhado com Platform Gateway (particionado)
- RPM: deve rodar em servidor separado de Controller e Hub
- Containerizado (growth): colocado no mesmo host; enterprise: nós dedicados

## Sistema Redis

| Modo | Quando usar | Nós necessários | HA |
|---|---|---|---|
| **Standalone** | Desenvolvimento, growth (não produção) | 1 | Não |
| **Cluster** | Produção enterprise | 6 (3 primários + 3 réplicas) | Sim |

```yaml
# Inventory: modo standalone (growth)
redis_mode=standalone

# Inventory: modo cluster (enterprise) — padrão em containerizado enterprise
# (não declarar redis_mode, o cluster é configurado automaticamente)
```

- Redis cluster: porta **6379** (dados) + **16379** (cluster bus)
- Dados Gateway e EDA são **particionados** — nenhum serviço acessa dados do outro
- Falha de nó primário → promoção automática de réplica

## PostgreSQL

- **AAP managed**: PostgreSQL 15 (gerenciado pelo instalador)
- **External (customer)**: PostgreSQL 15, 16 ou 17 com suporte a **ICU**
- Backup/restore nativo do AAP funciona apenas com PostgreSQL 15
- PostgreSQL 16/17 externos dependem de processos externos de backup
- Habilitar extensão **hstore** para o banco do Automation Hub
- Não colocar banco no mesmo nó que o Controller (degradação de performance)

### Variáveis de banco externo (inventory)
```ini
[all:vars]
postgresql_admin_username=postgres
postgresql_admin_password=<senha>

# Controller
automationcontroller_pg_host=<fqdn_db>
automationcontroller_pg_port=5432
automationcontroller_pg_database=awx
automationcontroller_pg_username=awx
automationcontroller_pg_password=<senha>

# Hub
automationhub_pg_host=<fqdn_db>
automationhub_pg_database=automationhub
# Hub e Controller DEVEM usar databases diferentes, mesmo que no mesmo servidor PostgreSQL
```

## Automation Mesh

- Overlay network baseada em **Receptor** (protocolo TCP 27199)
- Permite execução de playbooks distribuída geograficamente
- Tipos de nó:
  - **Control node**: executa tasks de controle do Controller
  - **Execution node**: executa playbooks (pode estar em DMZs remotas)
  - **Hop node**: relay de tráfego Receptor (mínimo CPU/RAM, não executa jobs)
  - **Hybrid node**: control + execution (padrão em growth topology)

### Portas Mesh
```
Controller → Execution node : TCP 27199 (Receptor)
Controller → Hop node       : TCP 27199 (Receptor)
Hop node   → Execution node : TCP 27199 (Receptor)
```

- Conexões são bidirecionais e FIPS-compliant
- `receptor_listener_port` configura a porta (padrão 27199)
- `receptor_peers` define os peers de cada nó
