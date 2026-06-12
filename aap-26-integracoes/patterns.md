# Patterns — Integrações, Hub e Controller

## Pattern: EDA + Terraform (IaC Event-Driven)

**Problema:** Terraform cria VM → automaticamente disparar configuração via Ansible.

```hcl
# Terraform: criar host e notificar EDA
resource "aap_host" "nova_vm" {
  inventory_id = aap_inventory.inv.id
  name         = aws_instance.vm.public_ip

  lifecycle {
    action_trigger {
      events  = [after_create]
      actions = [action.aap_eda_eventstream_post.configurar]
    }
  }
}

action "aap_eda_eventstream_post" "configurar" {
  config {
    job_template_name = "Configurar Nova VM"
    organization_name = "Default"
    event_stream_config = {
      url      = data.aap_eda_eventstream.stream.url
      username = var.eda_user
      password = var.eda_pass
    }
  }
}
```

```yaml
# EDA Rulebook — receber evento do Terraform
- name: Configurar VM criada via Terraform
  hosts: all
  sources:
    - eda.builtin.webhook:
        host: 0.0.0.0
        port: 5000
  rules:
    - name: Configurar nova VM
      condition: event.payload.job_template_name is defined
      action:
        run_job_template:
          name: "{{ event.payload.job_template_name }}"
          organization: "{{ event.payload.organization_name }}"
```

---

## Pattern: Vault como External Secret no Controller

**Problema:** Senhas e chaves de SSH não devem ser armazenadas no Controller — buscar do Vault.

```
1. Controller → Credential Types → Add → HashiCorp Vault Secret Lookup
   (configurar Vault Address + Token/AppRole)

2. Criar Machine Credential:
   → Username: root
   → Password: [ícone de chave] → selecionar Vault Lookup
     Metadata: secret_path=minhaapp/ssh, secret_key=password

3. Associar Machine Credential ao Job Template
   → No momento do job, o Controller busca a senha do Vault automaticamente
```

---

## Pattern: Pipeline CI/CD → Hub → Controller

**Problema:** Automatizar publicação de collections e EEs, e disparar testes via Controller.

```yaml
# .gitlab-ci.yml / GitHub Actions
stages:
  - build
  - publish
  - test

build_ee:
  script:
    - ansible-builder build --file ee.yml --tag hub.empresa.org/ci/meu-ee:$CI_COMMIT_SHORT_SHA

publish_ee:
  script:
    - podman login -u=robot+ci --password-stdin hub.empresa.org
    - podman push hub.empresa.org/ci/meu-ee:$CI_COMMIT_SHORT_SHA

trigger_tests:
  script:
    - |
      curl -X POST \
        -H "Authorization: Bearer $AAP_TOKEN" \
        -H "Content-Type: application/json" \
        -d '{"extra_vars": {"ee_tag": "'$CI_COMMIT_SHORT_SHA'"}}' \
        https://gateway.empresa.org/api/controller/v2/job_templates/15/launch/
```

---

## Pattern: Proteger Inventory com Ansible Vault + External Secret

**Problema:** Inventory tem senhas mas não pode ficar em texto claro no Git.

```bash
# Armazenar senhas no Vault, referenciar no inventory via awx-manage

# Opção 1: Ansible Vault no arquivo de credentials
ansible-vault encrypt credentials.yml
git add credentials.yml   # arquivo cifrado pode ir para Git

# Opção 2: awx-manage para injetar no Controller via script
sudo -u awx awx-manage inventory_import \
  --source=inventories/producao \
  --inventory-id=3 \
  --overwrite
```
