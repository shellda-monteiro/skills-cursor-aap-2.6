# Ch04 — Performance Tuning

## Tipos de Workload e Componentes Afetados

| Workload | Componentes impactados |
|---|---|
| Login de usuários (UI/SSO) | Platform Gateway (proxy + gRPC auth), web servers, database |
| API REST (CaC, CI/CD, scripts) | Gateway, gRPC auth, web server, database |
| Project sync (Git) | Control plane (somente), database |
| Jobs padrão (playbooks) | Execution plane, control plane (event processing), database |
| Workflow / Sliced / Bulk jobs | Job scheduling no control plane |
| EDA Activations | EDA hybrid nodes, Gateway (event streams), WebSocket, database |
| System jobs (cleanup) | Control plane, database |

## Tipos de Jobs

| Tipo | Descrição |
|---|---|
| **Standard** | Playbook contra inventory. Executa no execution plane. Gera job events (event volume cresce com verbosity). |
| **Sliced** | Split de inventory em fatias — paralelismo. |
| **Bulk** | Múltiplos jobs em uma requisição. |
| **Workflow** | Coordena múltiplos Job Templates em sequência/paralelo. |
| **System** | Manutenção interna (cleanup de job events). Roda no control plane. |

> **Importante sobre job events:** Um único debug task contra 1 host gera ~6 eventos em verbosity 1 e ~34 em verbosity 3. Alto verbosity degrada performance do Controller.

## Scaling: Vertical vs Horizontal

### Vertical (mais recursos por nó)

**Quando usar:**
- Alto uso de CPU/memória em nós existentes
- Primeiro passo antes de adicionar nós

**Limitações:**
- VMs com > 64 CPU e > 128 GB RAM podem não escalar linearmente
- Não automaticamente refletido — re-run do installer necessário para re-tuning
- Não overcommit recursos em hypervisor (CPU/RAM virtuais > físicos)

### Horizontal (mais nós/replicas)

**Quando usar:**
- Após validar configuração de tamanho de nó
- Precisar de alta disponibilidade
- Escalar capacidade de autenticação (mais gateways)

**Limitações:**
- Mais nós = mais conexões ao PostgreSQL → separar instâncias de DB por componente ao escalar muito
- Em OpenShift: mesh Envoy proxies fazem health checks cruzados — baseline de tráfego aumenta

## Regras de Scaling por Componente

| Componente | Escalar quando | Como |
|---|---|---|
| **Platform Gateway** | Autenticação lenta, muitos usuários simultâneos | Mais replicas/nós → mais capacity gRPC auth |
| **Automation Controller** | Jobs enfileirados, forks esgotados | Mais execution nodes (expand execution plane) |
| **Automation Hub** | Sync de collections lento, push de EEs lento | Mais workers (vertical) ou nós Hub |
| **EDA Controller** | Muitas ativações simultâneas, eventos atrasados | Mais workers EDA, mais nós EDA |
| **PostgreSQL** | Queries lentas, I/O alto | Separar instâncias por componente, aumentar IOPS |

## Configurações de Performance no Controller

### Forks

```
Controller UI → Settings → Jobs → Default Job Forks: N
```
Padrão: 5. Aumentar para ambientes com muitos hosts e jobs paralelos.

**Capacidade de forks por nó** = (RAM disponível ÷ memória por fork) e (CPUs × fator).
O installer calcula automaticamente — re-run o installer após vertical scaling.

### Desabilitar Live Updates na UI (reduzir carga)

```
Controller Settings → UI → UI_LIVE_UPDATES_ENABLED: false
```

Quando desabilitado: Job Detail page não atualiza automaticamente — precisa refresh manual.

### Token vs Basic Auth na API

Usar **Token authentication** para integrações API — melhor performance que Basic Auth:
```bash
# Criar token no Platform Gateway
# Gateway UI → Access → OAuth Tokens → Add

curl -H "Authorization: Bearer <token>" \
  https://gateway.exemplo.org/api/controller/v2/job_templates/
```

### Skip Audit Events no EDA (alto volume de eventos)

```
EDA UI → Rulebook Activations → [ativação] → Skip audit events: enabled
```

Quando habilitado: rules ainda disparam, mas o contador de eventos é atualizado a cada 300s (não em tempo real). Reduz carga no WebSocket e banco.

### Retenção de Job Events

**Regra:** reter o mínimo de dias possível. Dados antigos de jobs encarecem queries que fazem full table scans.

```
Controller Settings → System → Maximum days before deleting stdout: 30
Controller Settings → System → Maximum days to keep partial results: 14
```

## Sizing Reference para Topologia Growth vs Enterprise

### Growth (all-in-one)
- Máx recomendado: ~100 jobs/dia, ~200 hosts concorrentes
- Limitação: single point of failure em todos os componentes

### Enterprise (componentes separados)
- Controller: 16 GB RAM, 4 CPUs (por nó)
- Gateway: 8 GB RAM, 2 CPUs (por nó)
- Hub: 8 GB RAM, 2 CPUs (por nó)
- EDA: 8 GB RAM, 2 CPUs (por nó)
- DB: 16 GB RAM, 4 CPUs — **não compartilhar** — 3000+ IOPS

### Recomendação de migração Growth → Enterprise

Migrar para enterprise quando:
- Vertical scaling da growth fica impraticável (custo/disponibilidade)
- Requisitos de HA/DR não são atendidos pela growth
- Serviços precisam escalar independentemente
- Workloads consistentemente sobrecarregam a growth
