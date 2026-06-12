# Ch04 — Configuration as Code (CaC)

## O que é CaC no AAP

Gerenciar a configuração do Automation Controller (Job Templates, Inventários, Credenciais, Organizations, Teams, Usuários) usando arquivos YAML versionados em Git, aplicados via `ansible.platform` collection.

**Benefícios:**
- Histórico de mudanças com diff e rollback via Git
- Peer review e testes via CI/CD antes de aplicar em produção
- Recuperação rápida após falhas ou migrações
- Escalabilidade para novos clusters sem configuração manual

## ansible.platform Collection

A collection oficial para CaC no AAP 2.6. Substitui as collections anteriores `awx.awx` e `redhat_cop.controller_configuration`.

```bash
# Instalar
ansible-galaxy collection install ansible.platform

# Verificar módulos disponíveis
ansible-doc -l ansible.platform
```

**Módulos principais:**
```
ansible.platform.organization        # Criar/gerenciar organizações
ansible.platform.team                # Criar/gerenciar times
ansible.platform.user                # Criar/gerenciar usuários
ansible.platform.credential          # Criar/gerenciar credenciais
ansible.platform.credential_type     # Tipos de credencial customizados
ansible.platform.inventory           # Criar/gerenciar inventários
ansible.platform.host                # Adicionar hosts ao inventário
ansible.platform.project             # Sincronizar projetos Git
ansible.platform.job_template        # Criar Job Templates
ansible.platform.workflow_job_template  # Criar Workflow Templates
ansible.platform.schedule            # Criar agendamentos
ansible.platform.execution_environment  # Registrar EEs
```

## Setup Inicial do Ambiente CaC

```bash
# 1. Criar repositório Git
git init meu-aap-config

# 2. Encriptar senha do Gateway com Vault
ansible-vault encrypt_string '<gateway_password>' --name 'aap_password'
# Copiar o resultado para vars/all.yml

# 3. Estrutura de diretórios recomendada
meu-aap-config/
├── vars/
│   └── all.yml          # Variáveis de conexão e objetos AAP
├── playbooks/
│   └── configure_aap.yml  # Playbook principal
├── templates/             # Templates Jinja2 (opcional)
└── ansible.cfg
```

## vars/all.yml — Variáveis de Conexão

```yaml
---
# Conexão com Platform Gateway
aap_hostname: "https://gateway.exemplo.org"
aap_username: "admin"
aap_password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      <valor encriptado>
aap_validate_certs: true

# Organização
organizacao_nome: "TI-Infraestrutura"

# Credenciais que serão criadas
credenciais:
  - name: "vcenter-prod"
    credential_type: "VMware vCenter"
    organization: "{{ organizacao_nome }}"
    inputs:
      host: "vcenter.exemplo.org"
      username: "svc-ansible"
      password: !vault |
            $ANSIBLE_VAULT;1.1;AES256
            <valor encriptado>

# Job Templates
job_templates:
  - name: "Provisionar VM vCenter"
    organization: "{{ organizacao_nome }}"
    project: "automacoes-infra"
    playbook: "playbooks/criar_vm.yml"
    inventory: "inventario-producao"
    execution_environment: "ee-vmware:latest"
    credentials:
      - "vcenter-prod"
    ask_variables_on_launch: true
```

## playbooks/configure_aap.yml

```yaml
---
- name: Aplicar configuração CaC no AAP
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - ../vars/all.yml

  tasks:
    - name: Criar organização
      ansible.platform.organization:
        name: "{{ organizacao_nome }}"
        state: present
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"
        validate_certs: "{{ aap_validate_certs }}"

    - name: Criar credenciais
      ansible.platform.credential:
        name: "{{ item.name }}"
        credential_type: "{{ item.credential_type }}"
        organization: "{{ item.organization }}"
        inputs: "{{ item.inputs }}"
        state: present
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"
        validate_certs: "{{ aap_validate_certs }}"
      loop: "{{ credenciais }}"
      no_log: true   # não logar inputs com senhas

    - name: Criar Job Templates
      ansible.platform.job_template:
        name: "{{ item.name }}"
        organization: "{{ item.organization }}"
        project: "{{ item.project }}"
        playbook: "{{ item.playbook }}"
        inventory: "{{ item.inventory }}"
        execution_environment: "{{ item.execution_environment }}"
        credentials: "{{ item.credentials }}"
        ask_variables_on_launch: "{{ item.ask_variables_on_launch | default(false) }}"
        state: present
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"
        validate_certs: "{{ aap_validate_certs }}"
      loop: "{{ job_templates }}"
```

## Workflow CI/CD com CaC

```
1. Desenvolvedor cria/edita YAML de configuração
2. Abre Pull Request no repositório Git
3. CI executa ansible-lint e validação de sintaxe
4. Peer review aprova o PR
5. Merge para main → CI aplica no AAP de staging
6. Aprovação manual → CI aplica no AAP de produção
```

### Exemplo de pipeline (GitHub Actions)
```yaml
# .github/workflows/apply-cac.yml
name: Apply CaC to AAP
on:
  push:
    branches: [main]

jobs:
  apply:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: pip install ansible ansible.platform
      - name: Apply configuration
        run: |
          ansible-playbook playbooks/configure_aap.yml \
            --vault-password-file <(echo "${{ secrets.VAULT_PASSWORD }}")
        env:
          ANSIBLE_FORCE_COLOR: true
```

## Boas Práticas CaC

1. **Um arquivo por tipo de objeto** — `organizations.yml`, `credentials.yml`, `job_templates.yml`
2. **Nunca armazenar senhas em texto claro** — sempre Ansible Vault
3. **Usar `state: absent` com cuidado** — remover objetos em produção via CaC
4. **Tags por ambiente** — `dev`, `staging`, `production` para aplicar seletivamente
5. **`no_log: true`** em tasks com credenciais para evitar exposição nos logs
