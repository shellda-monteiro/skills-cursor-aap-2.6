# Cursor Skills — Red Hat Ansible Automation Platform 2.6

Skills para o [Cursor IDE](https://cursor.sh) com conteúdo extraído da documentação oficial do **Red Hat Ansible Automation Platform 2.6**, organizados para consulta direta durante desenvolvimento, implantação e consultoria.

---

## O que são Cursor Skills?

Skills são arquivos `SKILL.md` que ensinam o agente de IA do Cursor a responder perguntas técnicas usando conteúdo real — sem alucinações e sem precisar abrir PDFs. O agente carrega apenas os capítulos relevantes para cada pergunta.

---

## Skills disponíveis (branch `v2.6`)

| Skill | Conteúdo | Fonte |
|---|---|---|
| `aap-26-instalacao` | Arquitetura, topologias, instalação containerizada e Operator/OpenShift, rede e portas, PostgreSQL | ~6 guias |
| `aap-26-automacoes` | EDA Rulebooks, Decision Environments, Configuration as Code, Execution Environments, desenvolvimento | ~5 guias |
| `aap-26-operacao` | Gestão de acesso, autenticação (15 métodos), hardening/STIG, troubleshooting, logs | ~4 guias |
| `aap-26-mesh-performance` | Automation Mesh (nós hybrid/control/execution/hop), topologias, OCP Mesh, performance tuning, upgrade | ~5 guias |
| `aap-26-integracoes` | Terraform + Vault (HashiCorp), Private Automation Hub (coleções e EEs), awx-manage, backup/restore OCP | ~5 guias |
| `aap-26-controller-uso` | Job Templates, Workflows + Visualizer, Inventories dinâmicos (VMware/Satellite/AWS/Azure), Instance Groups, API REST e webhooks | ~2 guias |
| `aap-26-analytics-migracao` | Migração entre deployment types (RPM→Container, RPM→OCP), Automation Calculator (ROI), Savings Planner, Job Explorer | ~2 guias |
| `aap-26-dashboard` | Automation Dashboard (app local para métricas de até 3 instâncias AAP), clusters.yaml, token rotation, ROI, troubleshooting | 1 guia |
| `aap-26-dev-tools` | ansible-navigator (TUI/stdout, EEs, coleções, artifacts), Ansible Lightspeed Coding Assistant (VS Code, trial, on-premise), **Automation Intelligent Assistant** (chatbot na UI do AAP com MCP server — GA no OCP, Tech Preview no containerizado) | 3 guias + docs oficiais |
| `aap-26-self-service` | Self-Service Automation Portal (Helm/OCP, sincronização de templates, custom templates, RBAC, troubleshooting SSL) | 3 guias |
| `aap-26-developer-hub` | Ansible Plug-ins para Red Hat Developer Hub (Learn→Operate, Dev Spaces, software templates, instalação Helm/Operator) | 2 guias |

> Total: **~38 guias oficiais Red Hat** destilados em ~900 páginas de conteúdo estruturado.

---

## Instalação

### Instalar todos os skills de uma vez

```bash
git clone -b v2.6 git@github.com:laurobmb/skills-cursor-aap-2.6.git /tmp/aap-skills

cp -r /tmp/aap-skills/aap-26-instalacao         ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-automacoes         ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-operacao           ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-mesh-performance   ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-integracoes        ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-controller-uso     ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-analytics-migracao ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-dashboard          ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-dev-tools          ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-self-service       ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-developer-hub      ~/.cursor/skills/
```

### Instalar um skill específico (sparse checkout)

```bash
git clone -b v2.6 --filter=blob:none --sparse \
  git@github.com:laurobmb/skills-cursor-aap-2.6.git /tmp/aap-skills
cd /tmp/aap-skills
git sparse-checkout set aap-26-controller-uso
cp -r aap-26-controller-uso ~/.cursor/skills/
```

### Manter atualizado

```bash
cd /tmp/aap-skills
git pull origin v2.6
cp -r aap-26-* ~/.cursor/skills/
```

---

## Como usar no Cursor

Após instalar, abra o Cursor e use o agente normalmente. Os skills são carregados automaticamente quando o contexto é relevante, ou podem ser invocados explicitamente:

```
# Instalação e arquitetura
Usando aap-26-instalacao, quais são os requisitos mínimos de IOPS para o PostgreSQL?
Usando aap-26-instalacao, quais portas preciso liberar no firewall para o Automation Mesh?

# Automações
Usando aap-26-automacoes, como configurar um Rulebook para receber webhook do GLPI?
Usando aap-26-automacoes, como construir um EE customizado com coleções internas?

# Operação e segurança
Usando aap-26-operacao, como configurar LDAP no Platform Gateway?
Usando aap-26-operacao, quais variáveis de inventory habilitam FIPS?

# Mesh e performance
Usando aap-26-mesh-performance, como configurar um hop node para DMZ outbound-only?
Usando aap-26-mesh-performance, quais tipos de job afetam o scaling horizontal?

# Integrações
Usando aap-26-integracoes, como configurar o HashiCorp Vault como External Secret Provider?
Usando aap-26-integracoes, como sincronizar coleções do console.redhat.com para o Hub privado?

# Uso do Controller
Usando aap-26-controller-uso, como criar um workflow com aprovação humana antes do deploy?
Usando aap-26-controller-uso, como disparar um Job Template via API com extra_vars?
Usando aap-26-controller-uso, como configurar inventory dinâmico do VMware vCenter?

# Migração e analytics
Usando aap-26-analytics-migracao, qual o passo a passo para migrar de RPM para containerizado?
Usando aap-26-analytics-migracao, como calcular e apresentar o ROI da automação para um cliente?

# Dashboard local
Usando aap-26-dashboard, como conectar o Automation Dashboard a duas instâncias AAP?
Usando aap-26-dashboard, como resolver erro 401 na sincronização do dashboard?

# Ferramentas de desenvolvimento
Usando aap-26-dev-tools, como configurar o ansible-navigator para usar um EE corporativo?
Usando aap-26-dev-tools, como ativar o Ansible Lightspeed no VS Code?

# Self-service portal
Usando aap-26-self-service, como instalar o Self-Service Portal no OpenShift via Helm?
Usando aap-26-self-service, como filtrar apenas templates com label específica na sincronização?

# Developer Hub
Usando aap-26-developer-hub, como instalar os Ansible Plug-ins no RHDH existente?
Usando aap-26-developer-hub, qual a diferença entre o Self-Service Portal e os Ansible Plug-ins?
```

---

## Estrutura de cada skill

```
aap-26-<nome>/
├── SKILL.md              # Core: descrição, modelos mentais, índice de capítulos
├── chapters/             # Capítulos carregados sob demanda
│   ├── ch01-*.md
│   ├── ch02-*.md
│   └── ...
├── glossary.md           # Termos técnicos com definições precisas
├── patterns.md           # Padrões de decisão arquitetural com exemplos reais
└── cheatsheet.md         # Tabelas rápidas: portas, versões, variáveis, comandos
```

---

## Origem do conteúdo

Gerado a partir da documentação oficial Red Hat AAP 2.6 usando extração técnica com [docling](https://github.com/DS4SD/docling), preservando tabelas e blocos de código.

| Guia | Skill |
|---|---|
| Planning your installation | aap-26-instalacao |
| Containerized installation | aap-26-instalacao |
| Installing on OpenShift Container Platform | aap-26-instalacao |
| Tested deployment models | aap-26-instalacao |
| Using automation decisions (EDA) | aap-26-automacoes |
| Configuration as Code | aap-26-automacoes |
| Creating and using Execution Environments | aap-26-automacoes |
| Developing automation content | aap-26-automacoes |
| Access management and authentication | aap-26-operacao |
| Hardening and compliance | aap-26-operacao |
| Troubleshooting Ansible Automation Platform | aap-26-operacao |
| Automation Mesh for VM-based installations | aap-26-mesh-performance |
| Automation Mesh for Operator-based installations | aap-26-mesh-performance |
| Performance considerations | aap-26-mesh-performance |
| Upgrading / Migrating to AAP 2.6 (RPM) | aap-26-mesh-performance |
| Getting started with Hashicorp (Terraform + Vault) | aap-26-integracoes |
| Managing automation hub content | aap-26-integracoes |
| Managing containers in private automation hub | aap-26-integracoes |
| Configuring automation execution (advanced) | aap-26-integracoes |
| Using automation execution (Controller) | aap-26-controller-uso |
| Automation execution API overview | aap-26-controller-uso |
| Ansible Automation Platform migration | aap-26-analytics-migracao |
| Using automation analytics | aap-26-analytics-migracao |
| Using automation dashboard | aap-26-dashboard |
| Using content navigator (ansible-navigator) | aap-26-dev-tools |
| Ansible Lightspeed User Guide | aap-26-dev-tools |
| Ansible Lightspeed Release Notes | aap-26-dev-tools |
| Installing self-service automation portal | aap-26-self-service |
| Configuring self-service automation portal | aap-26-self-service |
| Using self-service automation portal | aap-26-self-service |
| Installing Ansible plug-ins for Red Hat Developer Hub | aap-26-developer-hub |
| Using Ansible plug-ins for Red Hat Developer Hub | aap-26-developer-hub |

---

## Contribuindo

Pull requests são bem-vindos para corrigir imprecisões ou adicionar conteúdo de novos guias do AAP 2.6.
