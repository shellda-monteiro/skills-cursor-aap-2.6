# Ch02 — Instalação dos Ansible Plug-ins no RHDH

## Pré-requisitos

| Requisito | Detalhe |
|---|---|
| **RHDH** | Instalado no OCP (via Helm chart ou Operator) |
| **Subscription** | AAP válida |
| **OCP** | Permissão para criar aplicações no projeto RHDH |
| **Acesso ao registry** | O RHDH deve acessar registry.redhat.io (ou registry interno para air-gap) |
| **Opcional** | Acesso a developers.redhat.com para Learning Paths |

## Dois Métodos de Entrega de Plug-ins

### OCI (Recomendado)

O RHDH puxa os plug-ins diretamente do `registry.redhat.io` como artefatos OCI na inicialização. Não requer downloads manuais.

```bash
# Criar secret de autenticação do registry no namespace do RHDH
oc create secret docker-registry rhdh-pull-secret \
  --docker-server=registry.redhat.io \
  --docker-username=<service_account_username> \
  --docker-password=<service_account_token> \
  -n <namespace_rhdh>
```

### HTTP Plug-in Registry (Air-gap)

Para ambientes sem acesso ao registry.redhat.io:

```bash
# 1. Baixar tarball dos plug-ins de access.redhat.com
# 2. Fazer deploy de um servidor HTTP interno com os arquivos
# 3. Configurar RHDH para usar o HTTP registry interno
```

## Adicionar Plug-ins ao Helm Chart (Instalação Helm)

Editar o `values.yaml` do RHDH para incluir os plug-ins Ansible:

```yaml
# values.yaml do RHDH (não do Self-Service Portal)
upstream:
  backstage:
    extraEnvVars:
      - name: LOG_LEVEL
        value: debug

    # Método OCI (recomendado)
    extraVolumes:
      - name: ansible-plugins
        ephemeral:
          volumeClaimTemplate:
            spec:
              accessModes:
                - ReadWriteOnce
              resources:
                requests:
                  storage: 2Gi

global:
  dynamic:
    includes:
      - dynamic-plugins.default.yaml
    plugins:
      - package: "./dynamic-plugins/dist/redhat-ansible-plugin"
        disabled: false
      - package: "./dynamic-plugins/dist/redhat-ansible-plugin-backend"
        disabled: false
```

## Criar ConfigMap de Configuração

```yaml
# ansible-plugins-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ansible-plugins-config
  namespace: <namespace_rhdh>
data:
  app-config.yaml: |
    ansible:
      creatorService:
        baseUrl: 'https://<RHDH_URL>'
        token: '<RHDH_SERVICE_ACCOUNT_TOKEN>'
      rhaap:
        baseUrl: 'https://gateway.empresa.org'          # AAP Platform Gateway
        token: '<AAP_API_TOKEN>'                        # token read do Controller
        checkSSL: true
      devSpaces:
        baseUrl: 'https://devspaces.empresa.org'        # (opcional)
      automationHub:
        baseUrl: 'https://hub.empresa.org'              # (opcional, Private Hub)
```

```bash
oc apply -f ansible-plugins-config.yaml

# Adicionar o ConfigMap ao Helm chart
helm upgrade <release> <chart> \
  --set-json 'upstream.backstage.extraAppConfig=[{"filename":"app-config.yaml","configMapRef":"ansible-plugins-config"}]' \
  -n <namespace_rhdh>
```

## Instalação via Operator

```
OCP Console → Operators → Red Hat Developer Hub → <instância RHDH>
→ Configuration → Dynamic Plugins
→ Add plugin:
    package: redhat-ansible-plugin
    disabled: false
→ Add plugin:
    package: redhat-ansible-plugin-backend
    disabled: false
→ Save (redeploy automático)
```

## Upgrade dos Plug-ins

```bash
# Helm: atualizar versão no values.yaml e fazer upgrade
helm upgrade <release> <chart> -f values.yaml -n <namespace>

# Operator: editar a instância RHDH → atualizar versão do plug-in → Save
```
