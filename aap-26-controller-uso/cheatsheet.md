# Cheatsheet — Automation Controller (Uso)

## Job Templates — Campos Chave

```
Inventory         → Prompt on Launch = sim (reutilizar em vários ambientes)
Credentials       → Prompt on Launch = sim
Variables         → Prompt on Launch = sim (sobrescrever via API/schedule)
Forks             → padrão 5; aumentar para patching em massa
Job Slice Count   → 1 = normal; >1 = distribuído (N sub-jobs)
Instance Groups   → restringe execução a nós específicos
Concurrent Jobs   → OFF por padrão; ON para templates paralelos
```

## API — Chamadas Essenciais

```bash
# Token
curl -u user:pass -X POST https://gw/api/gateway/v1/tokens/

# Lançar job
curl -H "Authorization: Bearer TOKEN" -X POST \
  https://gw/api/controller/v2/job_templates/<id>/launch/

# Status
curl -H "Authorization: Bearer TOKEN" \
  https://gw/api/controller/v2/jobs/<id>/

# Filtros
?status=failed    ?order_by=-created    ?page_size=100
```

## Inventory Plugins Suportados

| Ambiente | Plugin |
|---|---|
| VMware vCenter | `community.vmware.vmware_vm_inventory` |
| Red Hat Satellite | `theforeman.foreman.foreman` |
| AWS | `amazon.aws.aws_ec2` |
| Azure | `azure.azcollection.azure_rm` |
| GCP | `google.cloud.gcp_compute` |
| ServiceNow | `servicenow.itsm.now` |

> Arquivo de plugin: extensão `.now.yml` ou `.now.yaml` obrigatória

## Instance Groups — Hierarquia

```
Job Template  >  Inventory  >  Organization  >  default
(maior prioridade)                        (menor prioridade)
```

## Webhooks

```
Template → Enable Webhook → GitHub/GitLab
→ Webhook URL + Webhook Key → configurar no repositório
→ Push no repo → Controller lança o job automaticamente
→ Payload disponível como: awx_webhook_payload
```

## Workflow — Tipos de Nó

```
Job Template → executa playbook
Workflow Template → sub-workflow
Project Sync → atualiza Git
Inventory Sync → sincroniza fonte
Approval → pausa aguardando humano (timeout configurável)
```

## set_stats — Passar Variáveis entre Nós

```yaml
- set_stats:
    data:
      resultado: "{{ saida.json }}"
    per_host: false   # disponível downstream para todos os nós
```
