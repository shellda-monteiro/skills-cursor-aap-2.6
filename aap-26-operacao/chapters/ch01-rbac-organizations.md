# Ch01 — RBAC: Organizations, Teams e Permissões

## Estrutura de Organizations

Organizações são unidades de isolamento no AAP. Cada recurso (inventário, projeto, credencial, job template) pertence a uma organização.

```
Organization: "TI-Infraestrutura"
├── Teams:
│   ├── "Linux-Admins"     → pode executar job templates de patch
│   ├── "VMware-Ops"       → pode executar job templates de VM
│   └── "Readonly-Viewers" → só visualização
├── Users: (atribuídos via Teams ou diretamente)
├── Inventories: "Prod-Linux", "Prod-VMware"
├── Projects: "automacoes-infra", "automacoes-rede"
├── Job Templates: "Patch RHEL", "Criar VM", "Backup Rede"
└── Credentials: "satellite-api", "vcenter-prod", "vault-prod"
```

## Criando Organization via UI

1. Platform Gateway UI → Access → Organizations → Add
2. Preencher Name, Description
3. Adicionar Max Hosts (limitar escopo se necessário)

## Roles Disponíveis por Tipo de Recurso

### Job Template Roles
| Role | Permissão |
|---|---|
| `Admin` | CRUD + executar |
| `Execute` | Só executar |
| `Read` | Só visualizar |

### Inventory Roles
| Role | Permissão |
|---|---|
| `Admin` | CRUD completo |
| `Use` | Pode ser usado em Job Templates |
| `Read` | Só leitura |
| `Update` | Pode sincronizar |
| `Ad Hoc` | Pode executar comandos ad hoc |

### Project Roles
| Role | Permissão |
|---|---|
| `Admin` | CRUD completo |
| `Use` | Pode ser usado em Job Templates |
| `Read` | Só leitura |
| `Update` | Pode sincronizar SCM |

## Fluxo de Configuração RBAC Recomendado

```
1. Criar Organization (ex: "TI-Infraestrutura")
2. Criar Teams com escopo claro (ex: "Ops-Linux", "Dev-Ansible")
3. Atribuir usuários aos Teams
4. Criar recursos (inventários, projetos, credenciais, job templates)
5. Conceder Roles dos recursos aos Teams
```

## Criar e Gerenciar Teams via CaC

```yaml
# vars/rbac.yml
organizacao: "TI-Infraestrutura"

teams:
  - name: "Ops-Linux"
    organization: "{{ organizacao }}"
    description: "Time de operações Linux"
  - name: "Dev-Ansible"
    organization: "{{ organizacao }}"
    description: "Desenvolvedores de automação"

# Playbook CaC
- name: Criar teams
  ansible.platform.team:
    name: "{{ item.name }}"
    organization: "{{ item.organization }}"
    description: "{{ item.description }}"
    state: present
    controller_host: "{{ aap_hostname }}"
    controller_username: "{{ aap_username }}"
    controller_password: "{{ aap_password }}"
  loop: "{{ teams }}"
```

## Usuários: Tipos e Privilégios

| Tipo | Descrição |
|---|---|
| `Normal User` | Acesso apenas ao que lhe foi concedido via roles |
| `System Auditor` | Read-only em toda a plataforma |
| `System Administrator` | Superuser — acesso irrestrito |

**Regra:** Em produção, nunca usar `System Administrator` para uso diário. Criar usuários com permissões mínimas necessárias.

## Auditing e Activity Stream

O AAP mantém log de todas as mudanças via Activity Stream:

```
Platform Gateway UI → Activity Stream
# Ou via API
GET /api/gateway/v1/activitystream/

# Filtros úteis
?object_type=job_template   # mudanças em Job Templates
?actor=<username>           # ações de um usuário específico
?timestamp__gt=2024-01-01   # a partir de uma data
```

**Dados capturados:** timestamp, usuário, ação, objeto modificado, diff do que mudou.
