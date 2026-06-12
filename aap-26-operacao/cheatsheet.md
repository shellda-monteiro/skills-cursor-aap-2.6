# Cheatsheet — AAP 2.6 Operação e Segurança

## RBAC — Roles por Tipo de Recurso

| Recurso | Roles disponíveis |
|---|---|
| Organization | Admin, Member |
| Job Template | Admin, Execute, Read |
| Inventory | Admin, Use, Read, Update, Ad Hoc |
| Project | Admin, Use, Read, Update |
| Credential | Admin, Use, Read |
| Execution Environment | Admin, Read |

## Autenticação — Campos LDAP Essenciais

```
LDAP Server URI:   ldaps://ad.empresa.org:636
Bind DN:           CN=svc-ansible,OU=SA,DC=empresa,DC=org
User Search Base:  OU=Users,DC=empresa,DC=org
Group Search Base: OU=Groups,DC=empresa,DC=org
User Attr Map:     {"first_name": "givenName", "last_name": "sn", "email": "mail"}
```

## Autenticação — Campos SAML Essenciais

```
SP Entity ID:     https://gateway.empresa.org/
IdP SSO URL:      https://idp.empresa.org/sso
IdP Cert:         <certificado base64 do IdP>
Callback URL:     https://gateway.empresa.org/sso/callback/
```

## Troubleshooting — Comandos Rápidos

```bash
# Status containers
podman ps -a

# Logs componente
podman logs automation-controller --tail 100

# Testar API
curl -k https://gateway.empresa.org/api/gateway/v1/ping/
curl -k https://controller.empresa.org/api/v2/ping/

# Testar LDAP
ldapsearch -H ldaps://ad.empresa.org:636 -D "CN=svc,DC=empresa,DC=org" -w <senha> -b "OU=Users,DC=empresa,DC=org" "(sAMAccountName=usuario)"

# Testar porta Receptor
nc -zv controller.empresa.org 27199

# Testar PostgreSQL
nc -zv db.empresa.org 5432

# Bundle de suporte (RPM installer — containerizado: coletar logs via podman)
podman logs automation-controller > controller.log
podman logs automation-gateway > gateway.log

# Estado do Mesh
awx-manage list_instances

# OpenShift: status AAP
oc get aap -n aap-production
oc get pods -n aap-production
oc get pvc -n aap-production
```

## Backup e Restore (Containerizado)

```bash
# Backup (destino padrão: ~/backups)
ansible-playbook -i inventory ansible.containerized_installer.backup

# Backup com destino customizado
ansible-playbook -i inventory ansible.containerized_installer.backup \
  -e "backup_dir=/mnt/backup/"

# Restore
ansible-playbook -i inventory ansible.containerized_installer.restore

# Restore com arquivo específico
ansible-playbook -i inventory ansible.containerized_installer.restore \
  -e "restore_backup_file=/mnt/backup/aap-backup.tar.gz"
```

## Checklist Operação Segura

```
[ ] LDAP/SAML configurado e testado
[ ] Break-glass account criado e guardado em cofre
[ ] Backup automático agendado (mínimo diário)
[ ] Activity Stream habilitado e retido 90+ dias
[ ] Instance Groups segmentados por zona de segurança
[ ] Credenciais em Vault externo (sem senhas estáticas)
[ ] TLS em todos os endpoints (sem HTTP puro)
[ ] Upgrade testado em staging antes de produção
[ ] Runbook de rollback documentado
[ ] Monitoramento de containers/pods habilitado
```

## Erros Comuns e Fixes Rápidos

| Erro | Causa | Fix |
|---|---|---|
| `401 Unauthorized` no registry | Credenciais RHN inválidas | Verificar `registry_username/password` |
| `cdn04.quay.io refused` | Firewall bloqueando novo CDN | Liberar `cdn04-06.quay.io:443` |
| `Cannot store on NFS` | Podman home em NFS | Mover `container_storage_path` para disco local |
| `Hostname not resolved` | DNS não propagado | Verificar DNS + `/etc/hosts` em todos os nós |
| `LDAP login failed` | TLS/cert inválido | Importar CA raiz no truststore do Controller |
| `PVC Pending` (OCP) | StorageClass sem RWX | Provisionar StorageClass RWX ou usar S3/Azure |
| `27199 refused` | Firewall bloqueando Receptor | Liberar TCP 27199 entre Controller e Execution nodes |
