# Ch01 — Integração Terraform + Ansible Automation Platform

## Visão Geral dos Fluxos de Integração

| Fluxo | Descrição |
|---|---|
| **Ansible-initiated** | AAP executa playbook que chama módulos Terraform (`hashicorp.terraform`) |
| **Terraform-initiated** | Terraform usa o Provider AAP (`ansible/aap`) para criar inventory e disparar jobs |
| **EDA + Terraform** | Terraform Actions disparam Event Stream no EDA → EDA dispara Job Template |

## Coleções Necessárias

```yaml
# hashicorp.terraform — integração Ansible → Terraform (Ansible-initiated)
# Disponível no Automation Hub (Red Hat Certified)
collections:
  - name: hashicorp.terraform

# ansible/aap provider — Terraform → AAP (Terraform-initiated)
# Configurar no bloco terraform { required_providers }
terraform {
  required_providers {
    aap = {
      source = "ansible/aap"
    }
  }
}
```

## Fluxo 1: Ansible-Initiated (AAP executa Terraform)

### Credencial para `hashicorp.terraform`

```yaml
# Criar Credential Type customizado no Controller
# Input configuration:
fields:
  - id: terraform_token
    type: string
    label: HCP Terraform Token
    secret: true

# Injector configuration:
env:
  TF_TOKEN_app_terraform_io: '{{ terraform_token }}'
```

### Exemplo de playbook

```yaml
- name: Provisionar infraestrutura com Terraform
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Inicializar workspace Terraform
      hashicorp.terraform.plan:
        project_path: "{{ playbook_dir }}/terraform/"
        state: planned

    - name: Aplicar plano
      hashicorp.terraform.apply:
        project_path: "{{ playbook_dir }}/terraform/"
        state: present
        variables:
          region: "us-east-1"
          instance_type: "t3.medium"
```

## Fluxo 2: Terraform-Initiated (Terraform dispara AAP)

### Provider AAP — criar inventory e disparar job

```hcl
# Configurar provider
provider "aap" {
  host                  = var.aap_host
  username              = var.aap_username
  password              = var.aap_password
  insecure_skip_verify  = false
}

# Buscar organização e job template
data "aap_organization" "org" {
  name = "Default"
}

data "aap_job_template" "configure" {
  name              = "Configure VM"
  organization_name = "Default"
}

# Criar inventory e host
resource "aap_inventory" "inv" {
  name         = "Terraform Inventory"
  organization = data.aap_organization.org.id
}

resource "aap_host" "vm" {
  inventory_id = aap_inventory.inv.id
  name         = aws_instance.my_vm.public_ip
  variables = jsonencode({
    "ansible_ssh_retries": 10
  })

  # Disparar job após criar o host
  lifecycle {
    action_trigger {
      events  = [after_create]
      actions = [action.aap_job_launch.configure]
    }
  }
}

action "aap_job_launch" "configure" {
  config {
    inventory_id        = aap_inventory.inv.id
    job_template_id     = data.aap_job_template.configure.id
    wait_for_completion = true
  }
}
```

## Fluxo 3: Terraform + EDA (Event-Driven)

### Configurar Event Stream no AAP

```bash
# EDA UI → Event Streams → Create
# Nome: "TF Actions Event Stream"
# Tipo: Basic Auth (ou HMAC)
```

### Terraform dispara Event Stream no EDA

```hcl
# Buscar event stream
data "aap_eda_eventstream" "stream" {
  name = "TF Actions Event Stream"
}

resource "aap_host" "vm" {
  inventory_id = aap_inventory.inv.id
  name         = aws_instance.my_vm.public_ip

  lifecycle {
    action_trigger {
      events  = [after_create]
      actions = [action.aap_eda_eventstream_post.notify]
    }
  }
}

action "aap_eda_eventstream_post" "notify" {
  config {
    limit             = "all"
    template_type     = "job"
    job_template_name = "Configure New VM"
    organization_name = "Default"
    event_stream_config = {
      username = var.eda_stream_user
      password = var.eda_stream_password
      url      = data.aap_eda_eventstream.stream.url
    }
  }
}
```

### Rulebook EDA para receber o evento

```yaml
- name: Receber evento do Terraform
  hosts: all
  sources:
    - eda.builtin.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: Configurar VM criada pelo Terraform
      condition: event.payload.job_template_name == "Configure New VM"
      action:
        run_job_template:
          name: "Configure New VM"
          organization: "Default"
          job_args:
            extra_vars:
              target_host: "{{ event.payload.host_name }}"
```
