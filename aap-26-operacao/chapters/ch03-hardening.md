# Ch03 — Hardening e Compliance

## Princípios de Hardening AAP

1. **Princípio do mínimo privilégio** — usuários e service accounts com apenas as permissões necessárias
2. **Criptografia em trânsito** — TLS em todas as comunicações, sem HTTP puro em produção
3. **Rotação de credenciais** — integrar com CyberArk, HashiCorp Vault ou ESO
4. **Audit logging** — manter Activity Stream acessível e retido
5. **Inventory seguro** — nunca texto claro em Git (Ansible Vault obrigatório)

## Variáveis de Segurança no Inventory (Tabela Oficial — Containerizado)

Extraídas do Hardening Guide AAP 2.6:

| Variável (Containerizado) | Valor Recomendado | Descrição |
|---|---|---|
| `postgresql_disable_tls` | `false` | Se `true`, desabilita TLS no PostgreSQL gerenciado. Padrão: `false`. |
| `controller_pg_sslmode` | `verify-full` | mTLS entre Controller e DB. `verify-full` = obrigatório + verificar CN. |
| `gateway_pg_sslmode` | `verify-full` | mTLS entre Gateway e DB. |
| `hub_pg_sslmode` | `verify-full` | mTLS entre Hub e DB. |
| `eda_pg_sslmode` | `verify-full` | mTLS entre EDA e DB. |
| `controller_nginx_disable_https` | `false` | Se `true`, desabilita HTTPS no Controller. Manter `false` em produção. |
| `gateway_nginx_disable_https` | `false` | Se `true`, desabilita HTTPS no Gateway. |
| `hub_nginx_disable_https` | `false` | Se `true`, desabilita HTTPS no Hub. |
| `eda_nginx_disable_https` | `false` | Se `true`, desabilita HTTPS no EDA. |
| `controller_nginx_disable_hsts` | `false` | Se `true`, desabilita HSTS no Controller. |
| `gateway_nginx_disable_hsts` | `false` | Se `true`, desabilita HSTS no Gateway. |
| `hub_nginx_disable_hsts` | `false` | Se `true`, desabilita HSTS no Hub. |
| `eda_nginx_disable_hsts` | `false` | Se `true`, desabilita HSTS no EDA. |

## Variáveis de Certificados PKI (Tabela Oficial)

| Variável RPM | Variável Containerizada | Descrição |
|---|---|---|
| `custom_ca_cert` | `custom_ca_cert` | CA customizada — instalada no truststore do sistema |
| `web_server_ssl_cert` | `controller_tls_cert` | Certificado PKI do Controller |
| `web_server_ssl_key` | `controller_tls_key` | Chave privada do Controller |
| `automationhub_ssl_cert` | `hub_tls_cert` | Certificado do Hub |
| `automationhub_ssl_key` | `hub_tls_key` | Chave do Hub |
| `automationedacontroller_ssl_cert` | `eda_tls_cert` | Certificado do EDA |
| `automationedacontroller_ssl_key` | `eda_tls_key` | Chave do EDA |
| `postgres_ssl_cert` | `postgresql_tls_cert` | Certificado do PostgreSQL gerenciado |
| `postgres_ssl_key` | `postgresql_tls_key` | Chave do PostgreSQL gerenciado |
| — | `gateway_tls_cert` | Certificado do Platform Gateway |
| — | `gateway_tls_key` | Chave do Platform Gateway |

> **Nota multi-gateway com Load Balancer:** quando há múltiplos gateways atrás de LB, `gateway_tls_cert`/`gateway_tls_key` são compartilhados e o CN deve corresponder ao FQDN do LB. Para certificados individuais por nó, definir como **host variables** (não em `[all:vars]`).



## Configurações de Segurança na UI

```
Platform Gateway → Settings → Security:
├── ALLOW_OAUTH2_FOR_EXTERNAL_USERS: false   # bloquear tokens OAuth2 para LDAP users
├── CSRF_TRUSTED_ORIGINS: ['https://gateway.empresa.org']
└── SECURE_HSTS_SECONDS: 31536000           # HSTS 1 ano
```

## Gestão de Segredos com HashiCorp Vault

```yaml
# Credential Type: HashiCorp Vault Secret Lookup
# Configurar Vault Credential no Controller:
vault_addr: "https://vault.empresa.org"
vault_token: !vault |
      $ANSIBLE_VAULT;1.1;AES256...

# Usar em Job Templates via lookup
- name: Obter senha do banco
  set_fact:
    db_password: "{{ lookup('hashi_vault', 'secret/path/db password=token vault_addr=https://vault.empresa.org') }}"
```

## External Secrets Operator (ESO) — OpenShift

Para instalações Operator no OpenShift, integrar ESO para rotação automática de segredos:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: aap-controller-credentials
  namespace: aap-production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: aap-controller-credentials
    creationPolicy: Owner
  data:
    - secretKey: controller_password
      remoteRef:
        key: aap/controller
        property: admin_password
```

## Isolamento de Execução com Instance Groups

Segmentar execution nodes por nível de sensibilidade:

```
Instance Group "prod-critical"    → apenas Execution Nodes dedicados à produção
Instance Group "dev-sandbox"      → Execution Nodes isolados para desenvolvimento
Instance Group "dmz-network"      → Execution Nodes na DMZ para ativos de rede
```

```yaml
# Job Template: restringir a Instance Group específico
- ansible.platform.job_template:
    name: "Backup Rede"
    instance_groups:
      - "dmz-network"
```

## Compliance DISA STIG, CIS e FIPS — Considerações Específicas

### Requisito: noexec em file systems

Perfis CIS e DISA STIG frequentemente exigem `noexec` em sistemas de arquivo como `/tmp` e `/var`. **Problema:** Ansible usa esses diretórios para arquivos temporários durante a execução.

**Solução:**
- Discussão necessária com o auditor de segurança para waivar o controle `noexec` nos diretórios usados pelo Ansible nos nós gerenciados
- No host AAP: `$HOME/.ansible/tmp` e `/tmp` devem permitir execução

### Requisito: `user.max_user_namespaces = 0`

DISA STIG exige essa configuração SE containers Linux não forem necessários. Como o AAP **usa containers** (EEs, Podman), esse controle **deve ser desativado** (valor diferente de 0) para o AAP funcionar.

### Requisito: Interactive session timeout

DISA STIG (RHEL 8 V2R1 e RHEL 9 V2R2) exige logout por inatividade (ex: 15 min). Isso pode **interromper instalações** ou operações de backup/restore que levam mais de 1 hora.

**Solução temporária durante instalação/manutenção:**
```bash
# Aumentar timeout do systemd-logind durante operações longas
sudo loginctl set-property $USER IdleActionSec=7200  # 2 horas
```

### Compliance — Checklist CIS AAP

```
[ ] TLS habilitado em todos os endpoints (sem HTTP puro)
[ ] Certificados válidos (não auto-assinados) em produção
[ ] Ansible Vault para todos os segredos no inventory
[ ] Rotação automática de credenciais via Vault/CyberArk
[ ] Autenticação LDAP/SAML integrada (não usar apenas local)
[ ] Break-glass account documentado e guardado em cofre
[ ] Activity Stream habilitado e retido por >= 90 dias
[ ] Execution Nodes isolados por sensibilidade (Instance Groups)
[ ] Acesso SSH aos nós AAP restrito a IPs de gestão
[ ] Podman containers rodando como rootless (padrão)
[ ] PostgreSQL com TLS e `sslmode=verify-full`
[ ] Redis com TLS habilitado
[ ] Receptor (Mesh) com certificados assinados pela CA interna
[ ] Installation host dedicado — separado dos nós AAP
[ ] Inventory file protegido com Ansible Vault
[ ] STIG waiver documentado para noexec e user namespaces (se aplicável)
```

## Receptor — Segurança do Automation Mesh

O Receptor usa TLS para todas as comunicações. As configurações de certificado são gerenciadas automaticamente pelo instalador, mas podem ser customizadas:

```ini
# Inventory — receptor com CA customizada
receptor_tls_cert=/path/to/receptor.crt
receptor_tls_key=/path/to/receptor.key
receptor_ca_cert=/path/to/receptor-ca.crt

# Signing keys para verificar integridade de trabalhos
receptor_signing_private_key=/path/to/signing.key
receptor_signing_public_key=/path/to/signing.pub
```
