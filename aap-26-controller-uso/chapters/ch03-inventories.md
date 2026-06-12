# Ch03 — Inventories: Estáticos, Dinâmicos e Plugins

## Tipos de Inventory

| Tipo | Quando usar |
|---|---|
| **Estático** | Hosts fixos, pequeno volume, ambientes estáveis |
| **Dinâmico (fonte)** | Hosts em cloud/plataforma (VMware, AWS, Satellite, etc.) |
| **Constructed** | Filtrar/agrupar hosts de outros inventories via expressões Jinja2 |
| **Smart** | ~~Filtro por fatos armazenados~~ — **DEPRECIADO**, usar Constructed |

## Inventory Dinâmico — Fontes Suportadas

| Fonte | Plugin | Collection |
|---|---|---|
| **VMware vCenter** | `vmware.vmware_rest.vm_info` / `community.vmware` | community.vmware ou vmware.vmware |
| **VMware ESXI** | `vmware.vmware.vms` | vmware.vmware |
| **Red Hat Satellite 6** | `theforeman.foreman.foreman` | theforeman.foreman |
| **AWS EC2** | `amazon.aws.aws_ec2` | amazon.aws |
| **Azure** | `azure.azcollection.azure_rm` | azure.azcollection |
| **Google Cloud (GCE)** | `google.cloud.gcp_compute` | google.cloud |
| **OpenStack** | `openstack.cloud.openstack` | openstack.cloud |
| **ServiceNow** | `servicenow.itsm.now` | servicenow.itsm |
| **Outro Controller AAP** | plugin AAP | ansible.platform |
| **Script customizado** | qualquer executável | (usar projeto Git) |

## Configurar Inventory Dinâmico (VMware como exemplo)

```
Controller UI → Inventories → Add Inventory
→ Nome: "VMware Produção"
→ Organization: TI-Infraestrutura

Sources → Add source
→ Tipo: VMware vCenter
→ Credential: (tipo VMware — vcenter_hostname, username, password)
→ Update on launch: sim (sincroniza antes de cada job)
→ Overwrite on sync: sim
```

### Arquivo de plugin (para versão avançada via projeto Git)

```yaml
# vmware_inventory.now.yml  (extensão .now.yml ou .now.yaml obrigatória)
plugin: community.vmware.vmware_vm_inventory
hostname: "{{ vcenter_hostname }}"
username: "{{ vcenter_username }}"
password: "{{ vcenter_password }}"
validate_certs: false
with_nested_properties: true
properties:
  - config
  - guest
  - runtime
  - summary
filters:
  - runtime.powerState == "poweredOn"
keyed_groups:
  - key: config.guestId
    prefix: os
  - key: summary.runtime.powerState
    prefix: power
```

## Satellite (Foreman) — Inventory por Lifecycle Environment

```yaml
# satellite_inventory.now.yml
plugin: theforeman.foreman.foreman
url: https://satellite.empresa.org
username: admin
password: "{{ satellite_password }}"
validate_certs: false
want_facts: true
want_params: true
legacy_hostvars: true
keyed_groups:
  - key: foreman['lifecycle_environment_name']
    prefix: lifecycle
  - key: foreman['content_view_name']
    prefix: cv
  - key: foreman['location_name']
    prefix: loc
  - key: foreman['organization_name']
    prefix: org
```

Resultado: grupos como `lifecycle_Production`, `lifecycle_Development`, `loc_Datacenter_SP`

## ServiceNow — Inventory Dinâmico

```yaml
# servicenow_inventory.now.yml
plugin: servicenow.itsm.now
query:
  - os: = Linux Red Hat
  - os: = Windows Server 2022
keyed_groups:
  - key: os
    prefix: os
```

## Constructed Inventory — Filtrar e Agrupar

**Uso:** combinar hosts de múltiplos inventories com lógica de agrupamento avançada.

```yaml
# No campo Variables do Constructed Inventory:
plugin: constructed
strict: false
groups:
  # hosts RHEL em produção
  prod_rhel: "'rhel' in group_names and 'producao' in group_names"
  # hosts Windows
  windows: "ansible_os_family == 'Windows'"
compose:
  ansible_host: hostvars[inventory_hostname]['ip_address'] | default(inventory_hostname)
```

## Inventory Estático — Estrutura de Grupos

```ini
[producao]
web1.empresa.org
web2.empresa.org
db1.empresa.org

[development]
dev-web1.empresa.org

[webservers:children]
producao
development

[all:vars]
ansible_user=ansible
ansible_become=true
```

## Update on Launch vs Manual Sync

| Configuração | Quando usar |
|---|---|
| **Update on launch** | Inventory precisa sempre estar atualizado antes do job |
| **Scheduled sync** | Atualizar em intervalos regulares (menos overhead) |
| **Manual sync** | Ambientes controlados ou testes |

## Export de Scripts de Inventory Antigos

```bash
# Recuperar scripts customizados do banco (após upgrade de versão antiga)
sudo -u awx awx-manage export_custom_scripts --filename=meus_scripts.tar
mkdir meus_scripts
tar -xf meus_scripts.tar -C meus_scripts

# Transformar em projeto Git (para usar como SCM inventory source)
cd meus_scripts
git init && git add . && git commit -m "inventories migrados"
git push origin main
```
