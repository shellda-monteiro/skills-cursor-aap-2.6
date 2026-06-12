---
name: aap-26-operacao
description: >-
  Referência técnica AAP 2.6 para operação e segurança: gestão de acesso (RBAC, Organizations, Teams),
  autenticação (LDAP, SAML, OIDC, OAuth2, Microsoft Entra ID), hardening e compliance (CIS benchmarks),
  troubleshooting de instalação e runtime, e operações de dia-2.
  Use quando precisar configurar LDAP/SAML no Platform Gateway, gerenciar RBAC e permissões,
  aplicar hardening no AAP, diagnosticar falhas de instalação ou runtime, ou realizar
  operações de manutenção como backup, restore, upgrade.
disable-model-invocation: true
---

# AAP 2.6 — Operação, Acesso e Segurança

Referência extraída dos documentos oficiais: *Access management and authentication*, *Hardening and compliance*, *Troubleshooting Ansible Automation Platform*, e *Operating Ansible Automation Platform* (AAP 2.6).

## Hierarquia de Controle de Acesso (RBAC)

```
Platform (Global)
└── Organization
    ├── Teams
    │   └── Users
    ├── Inventory
    ├── Projects
    ├── Job Templates
    ├── Credentials
    └── Execution Environments
```

**Regras fundamentais:**
- Cada recurso pertence a uma Organization
- Permissões são concedidas via Roles a Users ou Teams
- System Administrator tem acesso irrestrito a tudo
- Organization Administrator gerencia apenas sua organização

## Tipos de Autenticação Suportados

| Tipo | Protocolo | Quando usar |
|---|---|---|
| Local | Banco interno | Dev/lab, fallback |
| LDAP | LDAP/LDAPS | Active Directory corporativo |
| SAML 2.0 | XML | SSO federado (Okta, ADFS) |
| OIDC genérico | OAuth2/OIDC | Keycloak, qualquer IdP OIDC |
| Microsoft Entra ID | OAuth2 | Azure AD |
| Google OAuth2 | OAuth2 | Google Workspace |
| TACACS+ | TACACS+ | Dispositivos de rede |

## Índice de Capítulos

| Arquivo | Conteúdo |
|---|---|
| [chapters/ch01-rbac-organizations.md](chapters/ch01-rbac-organizations.md) | Organizations, Teams, Users, Roles, permissões |
| [chapters/ch02-autenticacao.md](chapters/ch02-autenticacao.md) | LDAP, SAML, OIDC, Entra ID, OAuth2, autenticação plugável |
| [chapters/ch03-hardening.md](chapters/ch03-hardening.md) | CIS benchmarks, hardening, compliance, secrets |
| [chapters/ch04-troubleshooting.md](chapters/ch04-troubleshooting.md) | Diagnóstico de falhas: instalação, runtime, conectividade |
| [chapters/ch05-operacoes-dia2.md](chapters/ch05-operacoes-dia2.md) | Backup, restore, upgrade, scaling, manutenção |
| [glossary.md](glossary.md) | RBAC, Authenticator, Authenticator Map, OAuth2, LDAP, etc. |
| [patterns.md](patterns.md) | Padrões: LDAP mapping, SAML SSO, rotação de senhas, auditing |
| [cheatsheet.md](cheatsheet.md) | Referência rápida: LDAP vars, SAML config, troubleshooting commands |
