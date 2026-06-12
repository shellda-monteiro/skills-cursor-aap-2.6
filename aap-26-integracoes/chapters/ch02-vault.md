# Ch02 — Integração HashiCorp Vault

## Visão Geral

A coleção `hashicorp.vault` (Red Hat Certified, no Automation Hub) substitui a `community.hashi_vault`. Permite gerenciar segredos no Vault via playbooks e injetar credenciais de Vault em Job Templates via Credential Types customizados.

**Autenticação suportada:**
- `appRole` (role_id + secret_id via environment variables)
- `Token` (VAULT_TOKEN via environment variable)

## Criar Credential Type para Vault

### Token Authentication

```yaml
# Input configuration
fields:
  - id: vault_token
    type: string
    label: HashiCorp Vault Token
    secret: true
  - id: vault_addr
    type: string
    label: Vault Address

# Injector configuration
env:
  VAULT_TOKEN: '{{ vault_token }}'
  VAULT_ADDR: '{{ vault_addr }}'
```

### AppRole Authentication

```yaml
# Input configuration
fields:
  - id: vault_role_id
    type: string
    label: Vault AppRole Role ID
  - id: vault_secret_id
    type: string
    label: Vault AppRole Secret ID
    secret: true
  - id: vault_addr
    type: string
    label: Vault Address

# Injector configuration
env:
  VAULT_APPROLE_ROLE_ID: '{{ vault_role_id }}'
  VAULT_APPROLE_SECRET_ID: '{{ vault_secret_id }}'
  VAULT_ADDR: '{{ vault_addr }}'
```

**Procedimento:** Controller UI → Automation Execution → Infrastructure → Credential Types → Create

## Usar hashicorp.vault em Playbooks

```yaml
- name: Gerenciar segredos no Vault
  hosts: localhost
  gather_facts: false
  tasks:
    # Criar/atualizar segredo KV2
    - name: Criar segredo de aplicação
      hashicorp.vault.kv2_secret:
        url: "{{ lookup('env', 'VAULT_ADDR') }}"
        path: "myapp/config"
        data:
          api_key: "minha-chave-secreta"
          db_password: "senha-banco"

    # Ler segredo KV2
    - name: Ler segredo do banco
      hashicorp.vault.kv2_secret_info:
        url: "{{ lookup('env', 'VAULT_ADDR') }}"
        path: "myapp/config"
      register: vault_secret

    - name: Usar o segredo
      debug:
        msg: "DB Password: {{ vault_secret.secret.data.db_password }}"
```

### AppRole explícito (sem env vars)

```yaml
- name: Criar segredo com AppRole
  hashicorp.vault.kv2_secret:
    url: https://vault.empresa.org:8200
    auth_method: approle
    role_id: "{{ vault_role_id }}"
    secret_id: "{{ vault_secret_id }}"
    path: myapp/config
    data:
      api_key: "valor"
```

## Migração de community.hashi_vault

### Módulo KV1

```yaml
# Antes (community.hashi_vault)
- community.hashi_vault.vault_kv1_get:
    path: "secret/myapp"
    token: "{{ vault_token }}"

# Depois (hashicorp.vault)
- hashicorp.vault.kv1_secret_info:
    path: "secret/myapp"
    # token via VAULT_TOKEN env var — não passar como parâmetro
```

### Módulo KV2

```yaml
# Antes (community.hashi_vault)
- community.hashi_vault.vault_kv2_get:
    path: "secret/data/myapp"
    token: "{{ vault_token }}"

# Depois (hashicorp.vault)
- hashicorp.vault.kv2_secret_info:
    path: "myapp"  # sem prefix "secret/data/"
    # mount_point: "secret" (padrão)
```

## Vault como External Secret para Credentials do Controller

O AAP suporta Vault como **External Credential Provider** nativo — buscar segredos em tempo de execução sem armazenar no Controller:

```
Controller UI → Credentials → Add → HashiCorp Vault Secret Lookup
→ Vault Address: https://vault.empresa.org
→ Token: <token ou AppRole>
→ API Version: v2

# Ao criar uma Credential de Machine:
Password: {{source.vault.myapp/config.db_password}}
```

Isso permite que o Controller busque a senha do Vault **no momento do job**, sem armazenar em texto claro.
