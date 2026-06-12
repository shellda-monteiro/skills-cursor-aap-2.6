# Patterns — Automation Controller (Uso)

## Pattern 1: Template Genérico Multi-Ambiente

**Problema:** Mesmo playbook precisa rodar em dev, staging e produção com inventories e credenciais diferentes.

```
Job Template: "Deploy Aplicação"
  Inventory:    [Prompt on Launch = sim]  → usuário seleciona: inv-dev, inv-staging, inv-prod
  Credentials:  [Prompt on Launch = sim]  → usuário seleciona: cred-dev, cred-prod
  Variables:    [Prompt on Launch = sim]  → usuário pode passar {"ambiente": "producao"}
  Survey:
    - "Versão para deploy?" → variável: app_version
    - "Reboot após deploy?"  → variável: reboot_allowed (boolean)
```

**Benefício:** Um único template atende todos os ambientes — sem duplicação.

---

## Pattern 2: Pipeline de Deploy com Aprovação Humana

```
Workflow: "Deploy Produção com Aprovação"
  Nó 1: Job "Validar Pré-requisitos" (staging)
    → [success] Nó 2: Approval "Aprovação Deploy Prod" (timeout 30min)
      → [approved] Nó 3: Job "Deploy Produção"
        ├── [success] Nó 4: Job "Smoke Tests Produção"
        └── [failure] Nó 5: Job "Rollback Produção"
      → [denied/timeout] Nó 6: Job "Notificar Cancelamento"
    → [failure] Nó 7: Job "Notificar Falha Validação"
```

---

## Pattern 3: Patching em Massa com Job Slicing

**Problema:** Aplicar patches em 500 servidores RHEL via Satellite.

```yaml
# Job Template: "Patching RHEL Mensal"
job_slice_count: 10       # divide os 500 hosts em 10 fatias de 50
concurrent_jobs: true     # já habilitado automaticamente pelo slicing
instance_groups: [instance_group_execution]

# Survey:
#   "Errata para aplicar?" → variável: satellite_errata
#   "Reiniciar após patch?" → variável: reboot_allowed

# Playbook: usa satellite_errata e reboot_allowed
```

**Resultado:** 10 jobs rodando em paralelo → tempo de execução dividido por 10.

---

## Pattern 4: Inventory Dinâmico VMware + Grupo por Cluster

```yaml
# vmware_prod.now.yml (no repositório Git do projeto)
plugin: community.vmware.vmware_vm_inventory
hostname: "{{ vcenter_hostname }}"
username: "{{ vcenter_username }}"
password: "{{ vcenter_password }}"
validate_certs: false
filters:
  - runtime.powerState == "poweredOn"
  - config.template == false
keyed_groups:
  - key: config.guestId
    prefix: os
  - key: summary.runtime.host
    prefix: esxi_host
compose:
  ansible_host: guest.ipAddress
```

**Uso:** Job Template aponta para este inventory source → grupos `os_rhel8_64guest`, `os_windows9Server_64Guest` gerados automaticamente.

---

## Pattern 5: CI/CD GitLab → AAP via API

```bash
# .gitlab-ci.yml
deploy_staging:
  stage: deploy
  script:
    - |
      JOB=$(curl -s -k -X POST \
        -H "Authorization: Bearer $AAP_TOKEN" \
        -H "Content-Type: application/json" \
        -d "{\"extra_vars\": {\"version\": \"$CI_COMMIT_SHORT_SHA\", \"env\": \"staging\"}}" \
        "$AAP_URL/api/controller/v2/job_templates/$TEMPLATE_ID/launch/")
      JOB_ID=$(echo $JOB | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")
      echo "AAP Job: $AAP_URL/#/jobs/playbook/$JOB_ID"
      # Aguardar conclusão
      while true; do
        STATUS=$(curl -s -k -H "Authorization: Bearer $AAP_TOKEN" \
          "$AAP_URL/api/controller/v2/jobs/$JOB_ID/" | \
          python3 -c "import sys,json; print(json.load(sys.stdin)['status'])")
        echo "Status: $STATUS"
        [[ "$STATUS" == "successful" ]] && exit 0
        [[ "$STATUS" == "failed" || "$STATUS" == "error" ]] && exit 1
        sleep 10
      done
  only:
    - main
```

---

## Pattern 6: Instance Groups por Ambiente/Rede

```
Arquitetura:
  instance_group_producao  → exec-prod-1, exec-prod-2  (acesso à rede prod)
  instance_group_dmz       → exec-dmz-1                (acesso à DMZ)
  instance_group_dev       → exec-dev-1                (acesso à rede dev)

Uso:
  Job Template "Deploy Prod"  → Instance Groups: [instance_group_producao]
  Job Template "Config DMZ"   → Instance Groups: [instance_group_dmz]
  
  → Garantia de que jobs de produção NUNCA rodam em nós de dev
  → Isolamento de rede sem configuração extra no playbook
```
