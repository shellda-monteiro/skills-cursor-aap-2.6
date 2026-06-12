# Padrões e Decisões de Arquitetura — AAP 2.6

## Padrão 1: Containerizado vs Operator

**Use Containerizado quando:**
- Infra é RHEL puro (9.4+) sem OpenShift
- Controle total sobre VMs e storage é necessário
- Equipe prefere modelo de instalação via Ansible inventory
- Ambiente air-gapped com configuração de mirror local

**Use Operator (OpenShift) quando:**
- Cluster OpenShift já existe e está operacional
- Deseja lifecycle management automático via Operator Hub
- Escalabilidade horizontal nativa do OCP é necessária
- Storage RWX ou S3/Azure Blob disponível para o Hub

**Anti-padrão:** Usar o Operator sem ter StorageClass RWX disponível — causa falha no Automation Hub que é silenciosa até tentar fazer upload de collections.

---

## Padrão 2: Growth vs Enterprise

**Use Growth quando:**
- PoC, desenvolvimento ou ambiente de testes
- Organização iniciando com AAP (< 50 nós gerenciados)
- Baixo volume de jobs concorrentes
- Orçamento ou infra limitados

**Use Enterprise quando:**
- Produção com SLA de disponibilidade
- > 100 nós gerenciados ou jobs concorrentes frequentes
- Redundância de componentes é obrigatória
- Requer Redis cluster (6 nós) para HA

**Anti-padrão:** Deploy growth em produção de grande escala — Redis standalone é ponto único de falha para toda a plataforma.

---

## Padrão 3: Redis Standalone vs Cluster

| | Standalone | Cluster |
|---|---|---|
| Topologia | growth | enterprise |
| Nós | 1 | 6 (3 primário + 3 réplica) |
| HA | Não | Sim (failover automático) |
| `redis_mode` no inventory | `standalone` | (omitir, cluster é padrão) |
| Falha de nó | Plataforma para | Réplica promovida automaticamente |

**Regra:** Qualquer ambiente com SLA usa cluster. Redis standalone apenas para lab/dev.

---

## Padrão 4: PostgreSQL Interno vs Externo

**Use PostgreSQL interno quando:**
- Growth topology / testes / desenvolvimento
- Não há DBA ou banco corporativo disponível
- Simplicidade de operação é prioridade

**Use PostgreSQL externo quando:**
- Produção enterprise com políticas de backup corporativo
- Banco gerenciado por equipe de DBA separada
- Reaproveitamento de cluster PostgreSQL existente
- Requer PostgreSQL 16 ou 17 (recursos mais novos)

**Requisitos externos obrigatórios:**
- PostgreSQL 15, 16 ou 17
- Extensão ICU habilitada
- Extensão hstore habilitada (somente banco do Hub)
- Bases de dados separadas para cada componente
- Backup externo obrigatório para PostgreSQL 16/17

---

## Padrão 5: Certificados TLS

| Abordagem | Quando usar | Como |
|---|---|---|
| Auto-assinados (padrão) | Lab/dev apenas | Não configurar nada |
| CA customizada gera todos | PKI interna disponível | `ca_tls_cert` + `ca_tls_key` |
| Certificados individuais | CAs diferentes por serviço | `<service>_tls_cert/key` |
| CA raiz institucional | Adicionar ao trust store | `custom_ca_cert` |

**Anti-padrão:** Usar certificados auto-assinados em produção sem adicionar a CA ao trust store → falhas em integrações de API e clientes que validam TLS.

---

## Padrão 6: Inventory File — Segurança

**Correto:**
```bash
# Separar senhas em credentials.yml e encriptar com Vault
ansible-vault encrypt credentials.yml
ansible-playbook -i inventory -e @credentials.yml \
  --ask-vault-pass -K ansible.containerized_installer.install
```

**Errado:**
```ini
# NÃO fazer — senhas expostas no inventory
gateway_admin_password=minhasenha123
registry_password=minhasenharhn
```

**Anti-padrão:** Versionar inventory com senhas em texto claro no Git → exposição imediata de todas as credenciais da plataforma.

---

## Padrão 7: Execution Environments

**Use EE padrão Red Hat quando:**
- Collections certified são suficientes
- Sem dependências Python customizadas
- Agilidade de deploy é prioridade

**Construa EE customizado quando:**
- Dependências Python específicas (pyvmomi, netmiko, etc.)
- Collections internas ou de terceiros não certificadas
- Versão específica de collection necessária

```yaml
# execution-environment.yml
version: 3
dependencies:
  galaxy:
    collections:
      - name: community.vmware
        version: ">=4.0.0"
      - name: ansible.netcommon
  python:
    - pyvmomi>=8.0.0
    - netmiko>=4.0.0
  system:
    - gcc [platform:rpm]
```

```bash
# Build e push para Private Hub
ansible-builder build -t my-ee:latest
podman push my-ee:latest <hub-fqdn>/org/my-ee:latest
```

---

## Padrão 8: DNS e Hostnames

**Regras obrigatórias:**
- FQDNs resolvíveis em **todos os nós** antes do deploy
- **Sem underscores (`_`)** nos hostnames — Skopeo rejeita (Hub como registry)
- Hífens (`-`) são permitidos se não estiverem no início ou fim
- Usar formato consistente no inventory (todos FQDN ou todos short name, não misturar)
- **Não usar `localhost`** em instalações multi-nó

**Anti-padrão:**
```ini
# ERRADO — misturando formatos
[automationcontroller]
localhost ansible_connection=local
hostA
hostB.example.com
172.27.0.4
```
