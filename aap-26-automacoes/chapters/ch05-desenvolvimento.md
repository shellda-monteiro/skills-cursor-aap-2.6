# Ch05 — Desenvolvimento de Automações: Roles, Playbooks e Boas Práticas

## ansible-navigator — Ambiente de Desenvolvimento

Ferramenta TUI para desenvolvimento local dentro de Execution Environments:

```bash
# Instalar
pip3 install --user ansible-navigator

# Configurar (ansible-navigator.yml)
ansible-navigator:
  ansible-runner:
    artifact-dir: /tmp/navigator
  execution-environment:
    enabled: true
    image: meu-ee:latest
    pull:
      policy: missing
  mode: stdout
  playbook-artifact:
    enable: true
    save-as: /tmp/navigator/{playbook_name}-artifact.json

# Executar playbook dentro do EE
ansible-navigator run meu_playbook.yml \
  --execution-environment-image meu-ee:latest \
  --inventory inventario.yml \
  --mode stdout
```

## Estrutura de Role Ansible (Padrão AAP)

```
minha-role/
├── tasks/
│   ├── main.yml         # Ponto de entrada — inclui as subtasks
│   ├── criar_vm.yml     # Lógica de criação
│   └── validar.yml      # Validações pré/pós
├── defaults/
│   └── main.yml         # Variáveis com valores padrão
├── vars/
│   └── main.yml         # Variáveis fixas (não overridáveis)
├── handlers/
│   └── main.yml         # Handlers (restart, reload)
├── templates/
│   └── config.j2        # Templates Jinja2
├── files/
│   └── script.sh        # Arquivos estáticos
├── meta/
│   └── main.yml         # Metadados: autor, dependências, plataformas
└── README.md
```

### meta/main.yml (padrão Galaxy)
```yaml
galaxy_info:
  author: "Time de Infraestrutura"
  description: "Provisiona VMs no vCenter"
  company: "Empresa SA"
  license: "Apache-2.0"
  min_ansible_version: "2.14"
  platforms:
    - name: RHEL
      versions: ["9", "10"]
  galaxy_tags:
    - vmware
    - provisioning
dependencies: []
```

## Boas Práticas de Playbooks

### Idempotência
```yaml
# RUIM — não idempotente
- name: Adicionar linha ao arquivo
  shell: echo "config=true" >> /etc/app.conf

# BOM — idempotente
- name: Garantir configuração presente
  ansible.builtin.lineinfile:
    path: /etc/app.conf
    line: "config=true"
    state: present
```

### Tratamento de erros e rollback
```yaml
- name: Aplicar configuração com rollback
  block:
    - name: Fazer backup da configuração atual
      ansible.builtin.copy:
        src: /etc/app.conf
        dest: /etc/app.conf.backup
        remote_src: true

    - name: Aplicar nova configuração
      ansible.builtin.template:
        src: templates/app.conf.j2
        dest: /etc/app.conf
      notify: Reiniciar serviço

  rescue:
    - name: Restaurar backup em caso de falha
      ansible.builtin.copy:
        src: /etc/app.conf.backup
        dest: /etc/app.conf
        remote_src: true

  always:
    - name: Verificar status do serviço
      ansible.builtin.service_facts:
```

### Validações pré e pós-execução
```yaml
- name: Pre-task validation
  ansible.builtin.assert:
    that:
      - vm_nome is defined
      - vm_nome | length > 0
      - vm_cpu | int >= 1
      - vm_ram_gb | int >= 1
    fail_msg: "Parâmetros obrigatórios não fornecidos"

# ... tasks principais ...

- name: Post-validation — verificar VM criada
  community.vmware.vmware_guest_info:
    hostname: "{{ vcenter_host }}"
    username: "{{ vcenter_user }}"
    password: "{{ vcenter_password }}"
    name: "{{ vm_nome }}"
    datacenter: "{{ datacenter }}"
  register: vm_info
  failed_when: vm_info.instance.hw_power_status != "poweredOn"
```

## ansible-lint — Qualidade de Código

```bash
# Instalar
pip3 install --user ansible-lint

# Executar no projeto
ansible-lint .

# Executar em arquivo específico
ansible-lint playbooks/criar_vm.yml

# Arquivo de configuração .ansible-lint
profile: production
exclude_paths:
  - .git/
  - tests/
warn_list:
  - yaml[line-length]
```

**Regras importantes que o lint verifica:**
- `no-free-form` — usar módulos com parâmetros explícitos, não strings livres
- `name[casing]` — nomes de tasks em maiúscula
- `risky-file-permissions` — sempre especificar `mode` em `file`/`copy`
- `no-changed-when` — tasks `command`/`shell` precisam de `changed_when`

## Padrão de Variáveis Extra nos Job Templates

```yaml
# Variáveis que o usuário passa ao executar o Job Template
# Definidas em "Extra Variables" ou "Survey" no Job Template

# Para Criação de VM (vCenter)
vm_nome: "app-prod-01"
vm_cpu: 4
vm_ram_gb: 16
vm_disco_gb: 60
vm_network: "VLAN-Producao"
vm_datastore: "DS-SSD-Prod"
vm_template: "rhel10-template"
vm_datacenter: "DC-Principal"

# Para Patch Management (Satellite)
satellite_hostname: "satellite.exemplo.org"
errata_ids:
  - "RHSA-2024:1234"
  - "RHBA-2024:5678"
pre_patch_snapshot: true
post_patch_validate: true
```
