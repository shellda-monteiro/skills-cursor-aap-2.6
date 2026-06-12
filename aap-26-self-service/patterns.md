# Patterns — Self-Service Portal

## Pattern 1: Portal para Usuário de Negócio — Provisionar VM

**Cenário:** Equipe de TI cria portal onde gerentes de projeto podem provisionar VMs sem abrir ticket.

```
AAP Job Template: "Provisionar VM vCenter"
  Survey:
    - Nome da VM (text)
    - Ambiente: dev/staging/prod (choice)
    - CPU (integer, padrão: 2)
    - RAM em GB (integer, padrão: 4)
    - Sistema operacional: RHEL 8/9, Windows (choice)
  Label: "self-service"

Portal Sync (values.yaml):
  labels: ["self-service"]
  surveyEnabled: true

Resultado:
  → Gerente acessa portal → "Provisionar VM"
  → Preenche formulário simples
  → AAP executa o playbook
  → Gerente recebe e-mail com o IP da VM criada
```

---

## Pattern 2: Múltiplos Templates para o Mesmo Job (por Perfil)

```
Job Template AAP: "Deploy Aplicação" (ID: 55)

Custom Template 1: "Deploy Aplicação (Dev)"
  catalog-info.yaml:
    annotations:
      ansible.com/aap-job-template-id: "55"
    spec:
      parameters:
        - properties:
            ambiente: { enum: ["dev"] }  # apenas dev
            versao: { type: string }
  RBAC: time-desenvolvimento

Custom Template 2: "Deploy Aplicação (Produção)"
  catalog-info.yaml:
    annotations:
      ansible.com/aap-job-template-id: "55"
    spec:
      parameters:
        - properties:
            ambiente: { enum: ["producao"] }
            versao: { type: string }
            aprovado_por: { type: string }  # campo extra
  RBAC: time-release-management

→ Mesmo job, dois formulários, dois públicos, controle de acesso separado
```

---

## Pattern 3: Substituir Service Desk com Portal

```
Cenário atual:
  Usuário → abre ticket no GLPI → analista executa playbook manualmente → fecha ticket

Cenário com portal:
  Usuário → acessa portal → seleciona "Reset Senha AD" → preenche usuário
  → Portal lança Job Template → senha resetada em 30s → usuário notificado

Benefícios:
  ✓ Sem tickets manuais
  ✓ Auditoria automática (via Activity Stream do AAP)
  ✓ Usuário executa sem conhecer Ansible
  ✓ Permissão granular por template (usuário só vê o que pode executar)
```

---

## Pattern 4: Portal em Ambiente Restrito (Air-Gapped)

```bash
# 1. Mirror das imagens necessárias
skopeo copy \
  docker://registry.redhat.io/rhdh/rhdh-hub-rhel9:latest \
  docker://registry.empresa.org/rhdh/rhdh-hub-rhel9:latest

# 2. Criar ImageDigestMirrorSet no OCP
oc apply -f - <<'EOF'
apiVersion: config.openshift.io/v1
kind: ImageDigestMirrorSet
metadata:
  name: rhdh-mirrors
spec:
  imageDigestMirrors:
    - mirrors:
        - registry.empresa.org/rhdh
      source: registry.redhat.io/rhdh
EOF

# 3. Helm values apontando para registry interno
# upstream:
#   backstage:
#     image:
#       registry: registry.empresa.org
#       repository: rhdh/rhdh-hub-rhel9

# 4. Instalar normalmente
helm install self-service-portal ... -f values.yaml
```
