# Ch05 — Support Matrix de Upgrade e Migração

## Regras Gerais

1. **Upgrade in-place de RHEL major version NÃO é suportado** — migrar via backup/restore para novo ambiente RHEL
2. Caminhos de upgrade sempre seguem versões consecutivas (2.4 → 2.6 é suportado; não pular mais de uma major)
3. RPM installer será **removido no 2.7** (depreciado desde 2.5)

## Versões RHEL Suportadas por Deployment Type

| Deployment type | RHEL suportado |
|---|---|
| RPM 2.6 | RHEL 9 |
| Containerizado 2.6 | RHEL 9, RHEL 10 |
| OpenShift Container Platform 2.6 | Conforme OCP Life Cycle Policy |

## Caminhos de Upgrade RPM (2.4/2.5 → 2.6)

### 2.4 RHEL 8 → 2.6 RHEL 9 (RPM)
1. Backup do RPM 2.4 em RHEL 8
2. Restore para RHEL 9 com RPM 2.4 fresh install
3. Upgrade RPM 2.4 → 2.6

### 2.4 RHEL 9 → 2.6 RHEL 9 (RPM)
1. Upgrade direto RPM 2.4 → 2.6

### 2.5 RHEL 9 → 2.6 RHEL 9 (RPM)
1. Upgrade direto RPM 2.5 → 2.6

### 2.5 RHEL 8 → 2.6 RHEL 9 (RPM)
1. Backup RPM 2.5 em RHEL 8
2. Restore para RHEL 9 com RPM 2.5 fresh
3. Upgrade RPM 2.5 → 2.6

## Caminhos de Migração RPM → Container

### 2.4 RHEL 8 → Container 2.6 RHEL 9
1. Backup RPM 2.4 RHEL 8 → Restore para RPM 2.4 RHEL 9
2. Upgrade RPM 2.4 → 2.6
3. Migrate RPM 2.6 → Container 2.6

### 2.5 RHEL 9 → Container 2.6 RHEL 9
1. Upgrade RPM 2.5 → 2.6
2. Migrate RPM 2.6 → Container 2.6

### 2.x RPM/Container → Container RHEL 10
Para Container RHEL 10:
1. Chegar primeiro ao Container 2.6 em RHEL 9 (ver acima)
2. Backup Container 2.6 RHEL 9
3. Restore para RHEL 10 com Container 2.6 fresh install

## Caminhos de Migração para OpenShift

| Source | Target | Processo |
|---|---|---|
| RPM 2.4 RHEL 8 | OCP 2.6 | RPM 2.4 RHEL 8 → RPM 2.4 RHEL 9 → Upgrade RPM 2.6 → Migrate to OCP 2.6 |
| RPM 2.5 RHEL 9 | OCP 2.6 | Upgrade RPM 2.5 → 2.6 → Migrate to OCP 2.6 |
| Container 2.6 RHEL 9 | OCP 2.6 | Migrate Container 2.6 → OCP 2.6 |
| RPM 2.6 RHEL 9 | OCP 2.6 | Migrate RPM 2.6 → OCP 2.6 (direto) |

## Inventory de Upgrade RPM 2.4 → 2.6 (Growth)

Exemplo de inventory para upgrade de Controller único (2.4) para growth 2.6:

```ini
# Inventory para upgrade de RPM 2.4 single controller → 2.6 growth

[automationgateway]
gateway.exemplo.org       # NOVO em 2.6

[automationcontroller]
controller.exemplo.org

[automationcontroller:vars]
peers=execution_nodes

[execution_nodes]
exec.exemplo.org          # NOVO em 2.6

[automationhub]
hub.exemplo.org           # NOVO se não existia

[automationedacontroller]
eda.exemplo.org           # NOVO em 2.6

[database]
db.exemplo.org

[all:vars]
registry_username=<seu_RHN_user>
registry_password=<seu_RHN_password>
# ... demais variáveis do inventory 2.4 preservadas
```

## Mudanças de IAM no Upgrade para 2.6

O AAP 2.6 introduziu um novo sistema de autenticação no Platform Gateway. No upgrade, dados de IAM são migrados:

- **Usuários e grupos LDAP/SAML** configurados no Controller são migrados para o novo sistema de authenticators do Platform Gateway
- Tokens OAuth do Controller antigo precisam ser **recriados** — ver procedure em "Replacing Controller Tokens in AAP 2.6"
- Credential format: `https://<gateway>/api/controller` em vez de URL direta do Controller

## Mudanças de API no AAP 2.6

- Endpoint da API agora via **Platform Gateway**: `https://<gateway>/api/controller/v2/...`
- Compatibilidade backward: URLs diretas do Controller ainda funcionam (para instâncias migradas de 2.4)
- Novos endpoints do Gateway: `https://<gateway>/api/gateway/v1/...`
- WebSocket para jobs: via Gateway em vez de Controller direto

## Checklist de Pré-Upgrade

```
[ ] Backup completo do banco de dados atual
[ ] Inventory file do installer atual preservado
[ ] Versão do RHEL verificada (sem upgrade de major in-place)
[ ] Espaço em disco suficiente (15 GB no diretório de instalação)
[ ] Conexão com registry.redhat.io validada (ou bundle offline disponível)
[ ] Credenciais de registry (RHN) disponíveis no inventory
[ ] Timeout de sessão interativa aumentado (para instalações > 1 hora)
[ ] Janela de manutenção definida (VM/Container: downtime necessário)
```
