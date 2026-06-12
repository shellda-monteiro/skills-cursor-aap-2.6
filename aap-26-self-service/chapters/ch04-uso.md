# Ch04 — Uso do Portal: Templates, RBAC e Troubleshooting

## Login no Portal

```
1. Acessar https://<URL_DO_PORTAL>
2. Clicar em "Sign In"
3. Redireciona para login do AAP
4. Entrar com credenciais AAP
5. Autorizar o acesso do portal
→ Retorna ao portal autenticado
```

> **Acesso:** apenas usuários com conta no AAP conectado ao portal podem fazer login.

## Templates Auto-Gerados (Sincronizados do AAP)

Após a sincronização, cada Job Template do AAP aparece como um template no portal:
- Formulário idêntico ao survey do Job Template
- Passo a passo guiado para o usuário
- Usuário lança o job sem ver YAML ou o Controller

## Custom Self-Service Templates

Templates customizados permitem criar **formulários específicos** por tipo de usuário para o mesmo Job Template:

```
Exemplo: Job Template "Configurar Rede"
  → Custom template "Configurar Rede (Básico)"
      formulário simples, apenas IP e VLAN
      RBAC: time-rede-basico
  → Custom template "Configurar Rede (Avançado)"
      formulário completo com MTU, QoS, ACLs
      RBAC: time-rede-avancado
```

### Estrutura do Custom Template (arquivo no Git)

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Template
metadata:
  name: configurar-rede-basico
  title: Configurar Rede (Básico)
  description: Configuração básica de rede para usuários não-técnicos
  tags:
    - networking
    - self-service
  annotations:
    ansible.com/aap-job-template-id: "42"   # ID do Job Template no AAP
spec:
  owner: time-rede
  type: automation
  parameters:
    - title: Configurações Básicas
      required:
        - ip_address
        - vlan_id
      properties:
        ip_address:
          title: Endereço IP
          type: string
          description: IP do servidor a configurar
        vlan_id:
          title: VLAN
          type: integer
          description: Número da VLAN (1-4094)
  steps:
    - id: launch-job
      name: Executar automação
      action: ansible:launchJob
```

### Importar Custom Template do Git

```
Portal → Catalog → Register existing component
→ URL do arquivo catalog-info.yaml no repositório Git
→ Analyze → Import
```

## RBAC — Controle de Acesso por Template

```
Portal → Administration → Template Permissions
  → Selecionar template
  → Adicionar grupo/time com permissão: Execute ou View
  → Salvar
```

Apenas membros do time configurado podem **ver e executar** o template.

## Gerenciar EEs e Coleções no Portal

```
Portal → Execution Environments
  → Explorar EEs sincronizados do AAP
  → Ver coleções disponíveis em cada EE

Portal → Collections
  → Explorar coleções disponíveis
```

## Troubleshooting: Erro de SSL no Login

**Erro:** `Login failed; caused by Error: Failed to send POST request: fetch failed`

**Causa:** certificado SSL customizado ou self-signed no AAP.

```bash
# 1. Extrair o certificado CA do AAP
openssl s_client -showcerts -connect gateway.empresa.org:443 \
  </dev/null 2>/dev/null | openssl x509 -outform PEM > aap-ca-cert.pem

# 2. Criar ConfigMap com o certificado
oc create configmap custom-ca-bundle \
  --from-file=ca-bundle.crt=aap-ca-cert.pem \
  -n self-service-portal

# 3. Atualizar values.yaml para montar o CA
# upstream:
#   backstage:
#     extraVolumes:
#       - name: custom-ca
#         configMap:
#           name: custom-ca-bundle
#     extraVolumeMounts:
#       - name: custom-ca
#         mountPath: /etc/pki/ca-trust/source/anchors/
#         readOnly: true

# 4. Helm upgrade
helm upgrade self-service-portal ... -f values.yaml -n self-service-portal
```

> **Não recomendado em produção:** `checkSSL: false` no values.yaml desabilita a verificação SSL.

## Verificar Pods do Portal

```bash
oc get pods -n self-service-portal

# Pods esperados:
# self-service-portal-backstage-*   Running
# self-service-portal-postgresql-*  Running
```
