# Ch04 — Troubleshooting

## Coletar Logs do AAP

### Containerizado (VM/Podman)
```bash
# Verificar status de todos os containers
podman ps -a --format "table {{.Names}}\t{{.Status}}"

# Logs de um componente específico
podman logs automation-controller
podman logs automation-hub
podman logs automation-eda-server
podman logs automation-gateway
podman logs automation-postgresql

# Seguir logs em tempo real
podman logs -f automation-controller

# Gerar SOS report (VM-based)
sosreport
# Gera bundle completo com configurações, logs e diagnósticos
```

### OpenShift Operator — must-gather
```bash
# Coletar dados de todo o cluster
oc adm must-gather \
  --image=registry.redhat.io/ansible-automation-platform-25/aap-must-gather-rhel8 \
  --dest-dir ./aap-must-gather

# Coletar apenas de um namespace específico
oc adm must-gather \
  --image=registry.redhat.io/ansible-automation-platform-25/aap-must-gather-rhel8 \
  --dest-dir ./aap-must-gather \
  -- /usr/bin/ns-gather aap

# Se o cluster não acessa registry.redhat.io:
oc adm inspect

# Compactar para abrir chamado no suporte
tar cvaf must-gather.tar.gz ./aap-must-gather/
```

## Troubleshooting de Instalação

### Falha por usuário sem sudo
```
TASK [Check sudo privileges] FAILED
Error: user 'aapuser' is not in sudoers
```
**Fix:** Adicionar ao `/etc/sudoers.d/aapuser`: `aapuser ALL=(ALL) NOPASSWD: ALL`

### Falha por NFS no home directory
```
Error: cannot store images on NFS share
```
**Fix:** Mover `$HOME/.local/share/containers/` para disco local:
```bash
# inventory
container_storage_path=/data/containers
```

### Falha de DNS/FQDN
```
TASK [Generate certificates] FAILED
Could not resolve hostname controller.example.org
```
**Fix:** Verificar `/etc/hosts` e DNS em todos os nós antes de re-executar.

### Falha de IOPS/PostgreSQL
```
ERROR: could not write to file "pg_wal/..."
```
**Fix:** Verificar IOPS disponível: `ioping -c 10 /var/lib/postgresql`

### Falha de registry (imagens)
```
Error pulling image: registry.redhat.io: 401 Unauthorized
```
**Fix:** Verificar `registry_username`/`registry_password` no inventory.

### Falha quay.io CDN (novo endpoint)
```
Error: cdn04.quay.io: connection refused
```
**Fix:** Liberar no firewall os endpoints `cdn04.quay.io` a `cdn06.quay.io`.

## Diagnóstico de Runtime

### Containers parados
```bash
# Verificar por que parou
podman logs <container_name> --tail 50

# Reiniciar container específico
podman restart automation-controller

# Se falhar o restart, verificar recursos
df -h                    # espaço em disco
free -m                  # memória disponível
podman system df         # espaço usado pelo Podman
```

### API não responde
```bash
# Testar endpoint do Gateway
curl -k https://gateway.exemplo.org/api/gateway/v1/ping/

# Testar endpoint do Controller
curl -k https://controller.exemplo.org/api/v2/ping/
```

### Problemas de autenticação LDAP
```bash
# Testar bind LDAP manualmente
ldapsearch -H ldaps://ad.empresa.org:636 \
  -D "CN=svc-ansible,OU=ServiceAccounts,DC=empresa,DC=org" \
  -w "<senha>" \
  -b "OU=Users,DC=empresa,DC=org" \
  "(sAMAccountName=usuario_teste)"

# Habilitar debug LDAP
# Platform Gateway Settings → Authentication → LDAP → Log Level: DEBUG
```

### Falha em Job Templates
```bash
# Via API — ver detalhes do job falho
GET /api/controller/v2/jobs/<job_id>/
GET /api/controller/v2/jobs/<job_id>/stdout/

# Verificar se EE está acessível
podman pull <ee-image>

# Verificar credenciais usadas no job
GET /api/controller/v2/jobs/<job_id>/credentials/
```

### Automation Mesh — Nó desconectado
```bash
# Verificar estado dos nós do Mesh
awx-manage list_instances

# Verificar conectividade Receptor
receptor-cli --config /etc/receptor/receptor.conf status
nc -zv <controller_hostname> 27199

# Reativar nó manualmente
awx-manage register_instance --hostname=<execution_node>
```

## OpenShift — Troubleshooting do Operator

```bash
# Pod em CrashLoopBackOff
oc describe pod <pod-name> -n aap
oc logs <pod-name> -n aap --previous

# PVC travado em Pending
oc describe pvc <pvc-name> -n aap
# → verificar StorageClass disponível e provisioner

# Operator não reconcilia
oc get events -n aap --sort-by='.lastTimestamp' | tail -20
oc logs deployment/aap-operator-controller-manager -n aap
```

## Referência de Arquivos de Log (Containerizado)

| Componente | Caminho do log |
|---|---|
| Controller | `$HOME/aap/controller/logs/` |
| Hub | `$HOME/aap/hub/logs/` |
| EDA | `$HOME/aap/eda/logs/` |
| Gateway | `$HOME/aap/gateway/logs/` |
| Receptor | `$HOME/aap/receptor/` |
| PostgreSQL | `podman logs automation-postgresql` |
| Redis | `podman logs automation-redis` |
