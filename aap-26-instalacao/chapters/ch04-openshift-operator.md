# Ch04 — Instalação via Operator (OpenShift)

## Pré-Requisitos OpenShift

| Requisito | Detalhe |
|---|---|
| OpenShift versão | 4.x (compatível com AAP 2.6 Operator) |
| Permissão do usuário | `cluster-admin` |
| Namespace | Dedicado (NÃO usar `default`) |
| StorageClass para Hub | **ReadWriteMany (RWX)** ou object storage (S3/Azure Blob) |
| PostgreSQL | Interno (PVC) ou externo (PostgreSQL 15/16/17 com ICU) |

## Fluxo de Instalação (UI)

```
1. OpenShift Console → OperatorHub
2. Buscar "Ansible Automation Platform"
3. Install Operator → escolher namespace dedicado
4. Criar AnsibleAutomationPlatform Custom Resource
5. Acompanhar status: oc get aap -n <namespace>
```

## Instalação via CLI (oc)

```bash
# 1. Criar namespace dedicado
oc new-project aap-production

# 2. Criar OperatorGroup
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: aap-operator-group
  namespace: aap-production
spec:
  targetNamespaces:
    - aap-production
EOF

# 3. Criar Subscription
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ansible-automation-platform-operator
  namespace: aap-production
spec:
  channel: stable-2.6
  name: ansible-automation-platform-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF

# 4. Verificar instalação
oc get csv -n aap-production
oc get pods -n aap-production
```

## StorageClass RWX para Automation Hub

O Automation Hub **exige** armazenamento compartilhado ReadWriteMany para os arquivos de conteúdo (Collections, EEs). Opções:

### Opção 1: StorageClass RWX nativa do cluster
```yaml
# Verificar StorageClasses disponíveis
# oc get storageclass

# No Custom Resource do Hub, referenciar a StorageClass RWX
spec:
  hub:
    storage_type: file
    file_storage_storage_class: <nome-da-storageclass-rwx>
    file_storage_size: 100Gi
```

### Opção 2: Amazon S3 (recomendado para produção)
```yaml
spec:
  hub:
    storage_type: s3
    s3_secret_name: hub-s3-secret  # Secret com credenciais S3
```
```bash
# Criar o Secret S3
oc create secret generic hub-s3-secret \
  --from-literal=s3-access-key-id=<access_key> \
  --from-literal=s3-secret-access-key=<secret_key> \
  --from-literal=s3-bucket-name=<bucket> \
  --from-literal=s3-region=<region> \
  -n aap-production
```

### Opção 3: Azure Blob
```yaml
spec:
  hub:
    storage_type: azure
    azure_secret_name: hub-azure-secret
```

## Custom Resource — AnsibleAutomationPlatform

### Growth (mínimo funcional)
```yaml
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatform
metadata:
  name: aap
  namespace: aap-production
spec:
  gateway:
    replicas: 1
    admin_password_secret: aap-admin-password
  controller:
    replicas: 1
  hub:
    replicas: 1
    storage_type: s3
    s3_secret_name: hub-s3-secret
  eda:
    replicas: 1
```

### PostgreSQL Externo (ambas topologias)
```yaml
spec:
  gateway:
    database:
      host: db.example.com
      port: 5432
      name: automationgateway
      user: gateway
      password_secret: gateway-db-secret
      sslmode: require
  controller:
    database:
      host: db.example.com
      name: awx
      user: awx
      password_secret: controller-db-secret
```

## Platform Gateway no OpenShift

O Gateway é instalado primeiro como ponto de entrada e depois os demais componentes são linked.

```bash
# Verificar rota criada
oc get route -n aap-production

# Obter URL do Gateway
oc get aap aap -n aap-production -o jsonpath='{.status.gatewayURL}'

# Obter senha admin
oc get secret aap-admin-password -n aap-production \
  -o jsonpath='{.data.password}' | base64 -d
```

## Redis no Operator

- Por padrão, o Operator provisiona Redis via PVC
- Configurar Redis externo se necessário:
```yaml
spec:
  gateway:
    redis:
      external_hostname: redis.example.com
      external_port: 6379
```

## Troubleshooting Operator

```bash
# Status geral
oc get aap -n aap-production
oc describe aap aap -n aap-production

# Logs do Operator
oc logs -n aap-production deployment/aap-operator-controller-manager

# Verificar PVCs (Hub precisa estar Bound)
oc get pvc -n aap-production

# Problemas de StorageClass → verificar provisioner
oc get storageclass
oc describe pvc <nome-pvc> -n aap-production

# Deletar e recriar PVC em caso de problema
oc delete pvc <nome-pvc> -n aap-production
```

## CSRF e Timeout no Operator

```yaml
spec:
  gateway:
    extra_settings:
      - setting: CSRF_TRUSTED_ORIGINS
        value: "['https://gateway.example.com']"
    route_tls_termination_mechanism: Edge
    route_timeout: 300s   # aumentar para operações longas
```
