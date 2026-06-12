# Ch02 — Instalação via Helm no OpenShift

## Sequência de Pré-Instalação

### 1. Criar OAuth Application no AAP

```
AAP → Access Management → OAuth Applications → Create OAuth Application
  Name: self-service-portal
  Organization: <organização escolhida>
  Authorization grant type: Authorization code
  Client type: Confidential
  Redirect URIs: https://exemplo.com  ← placeholder, atualizar após deploy
→ Salvar: clientId e clientSecret
```

### 2. Habilitar criação de tokens OAuth para usuários externos

```
AAP → Settings → Platform Gateway → Edit platform gateway settings
  → Allow external users to create OAuth2 tokens: Enabled
  → Save
```

### 3. Criar API Token no AAP (para sincronização)

```
AAP → Access Management → API Tokens → Create API Token
  Description: self-service-portal-sync
  OAuth application: self-service-portal
  Scope: Write
→ Salvar o token gerado
```

### 4. Criar Secrets no OpenShift

```bash
# Namespace onde o portal será instalado
NAMESPACE=self-service-portal
oc new-project $NAMESPACE

# Secret de autenticação OAuth (clientId/clientSecret)
oc create secret generic aap-auth-secret \
  --from-literal=AUTH_AAP_CLIENT_ID=<clientId> \
  --from-literal=AUTH_AAP_CLIENT_SECRET=<clientSecret> \
  -n $NAMESPACE

# Secret do API Token (para sincronização)
oc create secret generic aap-backend-secret \
  --from-literal=AAP_TOKEN=<api_token> \
  -n $NAMESPACE
```

## Helm Values (values.yaml)

```yaml
global:
  aap:
    url: https://gateway.empresa.org   # URL do AAP Platform Gateway

catalog:
  providers:
    rhaap:
      baseUrl: https://gateway.empresa.org
      token:
        secretRef: aap-backend-secret
        secretKey: AAP_TOKEN
      orgs: "MinhaOrganizacao"          # organização AAP a sincronizar
      sync:
        orgsUsersTeams:
          schedule:
            frequency: { minutes: 60 }  # sincronizar usuários/times a cada hora
            timeout: { minutes: 15 }
        jobTemplates:
          enabled: true
          schedule:
            frequency: { minutes: 60 }
            timeout: { minutes: 15 }

auth:
  providers:
    aap:
      production:
        clientId:
          secretRef: aap-auth-secret
          secretKey: AUTH_AAP_CLIENT_ID
        clientSecret:
          secretRef: aap-auth-secret
          secretKey: AUTH_AAP_CLIENT_SECRET
```

## Instalar via Helm

```bash
# Adicionar o repositório Helm do portal (via OpenShift Helm catalog)
# Alternativa: OpenShift Console → Helm → Helm Charts → Self-Service Automation Portal

# Instalar via CLI
helm install self-service-portal \
  openshift-helm-charts/ansible-automation-platform-self-service \
  -f values.yaml \
  -n self-service-portal

# Verificar status
helm status self-service-portal -n self-service-portal
oc get pods -n self-service-portal
```

## Pós-instalação: Atualizar Redirect URI no AAP

```bash
# Obter a URL de deploy do portal
oc get route -n self-service-portal

# Atualizar no AAP:
# Access Management → OAuth Applications → self-service-portal
# Redirect URIs: https://<URL_DO_PORTAL>/api/auth/aap/handler/frame
```

## Instalação em Ambiente Air-Gapped

Para OCP sem acesso à internet:

```bash
# 1. Mirror das imagens para registry interno
skopeo copy docker://registry.redhat.io/rhdh/... \
  docker://registry.empresa.org/rhdh/...

# 2. Configurar ImageContentSourcePolicy ou ImageDigestMirrorSet no OCP
# 3. Ajustar values.yaml para apontar para o registry interno
# 4. Instalar normalmente via helm
```
