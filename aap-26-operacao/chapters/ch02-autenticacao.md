# Ch02 — Autenticação: LDAP, SAML, OIDC, OAuth2

## Métodos de Autenticação Suportados (AAP 2.6)

O AAP 2.6 suporta os seguintes métodos via sistema de autenticação plugável do Platform Gateway:

| Método | Quando usar |
|---|---|
| **Local** | Conta local no AAP (padrão, sempre disponível) |
| **LDAP** | Active Directory / OpenLDAP com bind + busca de grupos |
| **SAML 2.0** | IdPs corporativos (Keycloak, ADFS, Okta) |
| **Generic OIDC** | Qualquer IdP compatível com OpenID Connect |
| **Microsoft Entra ID** (Azure AD) | Azure OAuth2/OIDC |
| **Google OAuth2** | Contas Google / Google Workspace |
| **Keycloak** | Keycloak/RHSSO direto |
| **GitHub** | Contas pessoais GitHub |
| **GitHub Organization** | Membros de uma organização GitHub |
| **GitHub Team** | Membros de um time GitHub |
| **GitHub Enterprise** | GitHub Enterprise Server |
| **GitHub Enterprise Organization** | Org no GitHub Enterprise |
| **GitHub Enterprise Team** | Time no GitHub Enterprise |
| **TACACS+** | Controle de acesso via protocolo TACACS+ |
| **RADIUS** | Autenticação via servidor RADIUS |

> **Dica:** Múltiplos autenticadores podem ser configurados em paralelo com ordem de prioridade. O primeiro que autenticar o usuário com sucesso é usado.

```
Usuario faz login
        │
        ▼
Platform Gateway verifica Authenticators (ordem configurável)
        │
        ├─ Local Auth?     → verificar senha interna
        ├─ LDAP?           → bind LDAP + busca de grupos
        ├─ SAML?           → verificar assertion
        └─ OIDC/Entra ID?  → verificar token
        │
        ▼
Authenticator Maps: mapear grupos/atributos → Org/Team/Role no AAP
```

## Configurar LDAP (Active Directory)

**Platform Gateway UI → Access → Authenticators → Add → LDAP**

### Campos obrigatórios

| Campo | Exemplo | Descrição |
|---|---|---|
| `LDAP Server URI` | `ldaps://ad.empresa.org:636` | URI com LDAPS recomendado |
| `Bind DN` | `CN=svc-ansible,OU=ServiceAccounts,DC=empresa,DC=org` | Service Account |
| `Bind Password` | `<senha>` | Senha do Service Account |
| `User Search` | `OU=Users,DC=empresa,DC=org` | Base de busca de usuários |
| `User DN Template` | `sAMAccountName=%(user)s,OU=Users,DC=empresa,DC=org` | (opcional) |
| `User Attr Map` | `{"first_name": "givenName", "last_name": "sn", "email": "mail"}` | Mapeamento de atributos |
| `Group Search` | `OU=Groups,DC=empresa,DC=org` | Base de busca de grupos |
| `Group Type` | `GroupOfNamesType` ou `MemberDNGroupType` | Tipo de grupo AD |

### Authenticator Map LDAP → Org/Team

Após criar o Authenticator, criar Authenticator Maps:

```
Map 1: AD Group "CN=Ansible-Admins,OU=Groups,DC=empresa,DC=org"
       → Organization: "TI-Infraestrutura", Role: Admin

Map 2: AD Group "CN=Ansible-Readers,OU=Groups,DC=empresa,DC=org"
       → Organization: "TI-Infraestrutura", Role: Member
```

### Importar CA Raiz para LDAPS
```bash
# No host do Controller, adicionar CA ao truststore
sudo cp empresa-root-ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust

# Ou via inventory (instalação)
custom_ca_cert=/path/to/empresa-root-ca.crt
```

## Configurar SAML 2.0

**Platform Gateway UI → Access → Authenticators → Add → SAML**

### Campos SAML

| Campo | Descrição |
|---|---|
| `SP Entity ID` | ID do AAP como Service Provider (ex: `https://gateway.empresa.org/`) |
| `IdP Entity ID` | ID do Identity Provider |
| `IdP SSO URL` | URL de login do IdP |
| `IdP x509 Cert` | Certificado de assinatura do IdP |
| `SP Public Cert` | Certificado do AAP (gerado ou fornecido) |
| `SP Private Key` | Chave privada do AAP |

### Authenticator Map SAML → Org

```
Atributo SAML: memberOf = "ansible-admins"
→ Organization: "TI-Infraestrutura", Role: Admin
```

### Login transparente SAML
Para redirecionar automaticamente para o IdP (sem exibir tela de login local):
- Platform Gateway → Authenticators → SAML → Enable `Redirect to SSO`

## Configurar Microsoft Entra ID (Azure AD)

**Platform Gateway UI → Access → Authenticators → Add → Azure AD**

### Pré-requisitos no Azure
1. Registrar aplicativo no Entra ID
2. Callback URL: `https://gateway.empresa.org/sso/callback/`
3. Permissões: `openid`, `profile`, `email`, `User.Read`, `GroupMember.Read.All`

### Campos no AAP

| Campo | Origem no Azure |
|---|---|
| `OIDC Key (Application ID)` | Application (client) ID |
| `OIDC Secret` | Client secret value |
| `OIDC URL` | `https://login.microsoftonline.com/<tenant-id>/v2.0` |

## OAuth2 Tokens para Integrações de API

Para automações externas que precisam chamar a API do AAP:

```bash
# Criar Personal Access Token
# Controller UI → User → Tokens → Add

# Usar token na API
curl -H "Authorization: Bearer <token>" \
  https://gateway.empresa.org/api/controller/v2/job_templates/

# Via ansible.platform collection (CaC)
controller_oauthtoken: "<token>"
# (em vez de username/password)
```

## Configurar Timeouts de Sessão

```
Platform Gateway UI → Settings → Authentication
TOKEN_EXPIRY_SECONDS: 28800    # 8 horas (padrão)
SESSION_COOKIE_AGE: 1800       # 30 min de inatividade
```

## Local Authenticator — Enable/Disable

**Atenção:** Desabilitar o autenticador local bloqueia login com usuário/senha se o LDAP/SAML estiver inacessível.

- Manter local habilitado com um usuário de emergência (break-glass account)
- Documentar credenciais do break-glass account em cofre seguro
