# Cheatsheet — Self-Service Automation Portal

## Pré-requisitos

```
✓ OCP com Helm 3.10+
✓ oc CLI instalado
✓ AAP 2.6 com permissão para criar OAuth Application
✓ Subscription AAP (RHDH incluído, sem custo adicional)
```

## Sequência de Instalação

```
1. AAP: criar OAuth Application (grant: authorization code, confidential)
2. AAP: habilitar "Allow external users to create OAuth2 tokens"
3. AAP: criar API Token (scope: write)
4. OCP: criar secrets (oauth + api token)
5. Helm: configurar values.yaml
6. helm install ... -f values.yaml -n self-service-portal
7. AAP: atualizar Redirect URI na OAuth Application
```

## Secrets OCP

```bash
oc create secret generic aap-auth-secret \
  --from-literal=AUTH_AAP_CLIENT_ID=<clientId> \
  --from-literal=AUTH_AAP_CLIENT_SECRET=<clientSecret> -n self-service-portal

oc create secret generic aap-backend-secret \
  --from-literal=AAP_TOKEN=<api_token> -n self-service-portal
```

## values.yaml — Chaves principais

```yaml
global.aap.url: https://gateway.empresa.org
catalog.providers.rhaap.orgs: "MinhaOrganizacao"
catalog.providers.rhaap.sync.jobTemplates.labels: ["self-service"]
catalog.providers.rhaap.sync.jobTemplates.surveyEnabled: true
```

## Filtros de Template

| Filtro | Efeito |
|---|---|
| `labels: []` ou omitido | Todos os templates |
| `labels: ["self-service"]` | Apenas templates com essa label |
| `surveyEnabled: true` | Apenas templates COM survey |
| `surveyEnabled: false` | Apenas templates SEM survey |

## Troubleshooting

```bash
# Erro SSL no login
openssl s_client -showcerts -connect gateway:443 </dev/null | \
  openssl x509 -outform PEM > aap-ca.pem
oc create configmap custom-ca-bundle --from-file=ca-bundle.crt=aap-ca.pem
# Adicionar ao values.yaml: extraVolumes + extraVolumeMounts
helm upgrade ...

# Verificar pods
oc get pods -n self-service-portal

# Jobs de sincronização presos
# Portal → Administration → Synchronize → forçar sync manual
```
