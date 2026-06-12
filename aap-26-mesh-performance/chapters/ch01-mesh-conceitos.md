# Ch01 — Automation Mesh: Conceitos e Tipos de Nós

## O que é Automation Mesh

Automation mesh é uma rede overlay que distribui execução de automação entre workers dispersos geograficamente, usando conexões peer-to-peer via redes existentes.

**Características:**
- Capacidade de cluster dinâmica — adicionar/remover nós com mínimo downtime
- Separação do **control plane** e **execution plane** — escalar independentemente
- Comunicação bi-direcional, multi-hop e compatível com FIPS
- TLS em toda comunicação — tráfego externo sempre criptografado

## Tipos de Nós

### Control Plane

| Tipo | Descrição | Jobs executados |
|---|---|---|
| **Hybrid** (padrão) | Default para nós do Controller. Executa runtime do Controller + automação | Tudo: project updates, management jobs, playbooks |
| **Control** | Apenas serviços de controle. Sem execução de playbooks | Project updates, inventory updates, system jobs (não playbooks de usuário) |

### Execution Plane

| Tipo | Descrição | Função |
|---|---|---|
| **Execution** (padrão) | Roda jobs com `ansible-runner` em Podman isolation | Execução de playbooks para usuário |
| **Hop** | Roteia tráfego para outros execution nodes — análogo a jump host | Relay — **não executa** automação |

> **Importante:** Control nodes são separados para ambientes enterprise onde se quer isolar a capacidade de execução. Hybrid nodes são mais simples mas combinam controle + execução.

## Variáveis de Tipo de Nó

### RPM-based
```ini
[automationcontroller]
ctrl.exemplo.org node_type=control    # apenas controle
ctrl2.exemplo.org node_type=hybrid    # padrão se omitido

[execution_nodes]
exec.exemplo.org                      # execution (padrão)
hop.exemplo.org node_type=hop         # hop node
```

### Containerizado (usar receptor_type em vez de node_type)
```ini
[automationcontroller]
ctrl.exemplo.org receptor_type=control

[execution_nodes]
exec.exemplo.org receptor_type=execution
hop.exemplo.org  receptor_type=hop
```

> **CRÍTICO:** Em instalações containerizadas, usar `receptor_type=` e `receptor_peers=` (não `node_type=` e `peers=`).

## Peering

Peers definem conexões nó-a-nó. Após estabelecido, o peering permite comunicação **bidirecional**.

```ini
# RPM: usar peers=
[automationcontroller]
ctrl.exemplo.org peers=execution-node-1.exemplo.org

[execution_nodes]
execution-node-1.exemplo.org peers=execution-node-2.exemplo.org
execution-node-2.exemplo.org
```

```ini
# Containerizado: usar receptor_peers= (lista de hostnames separada por vírgula)
# NÃO usar nomes de grupos de inventário — apenas hostnames explícitos
[automationcontroller]
ctrl.exemplo.org receptor_peers=exec1.exemplo.org,exec2.exemplo.org

[execution_nodes]
exec1.exemplo.org
exec2.exemplo.org
```

### Peering em grupo (RPM)
```ini
[automationcontroller:vars]
node_type=control
peers=execution_nodes    # peer com TODOS os nós do grupo execution_nodes
```

## TLS do Receptor (Automation Mesh)

O Receptor usa TLS para toda comunicação. Certificados são gerenciados pelo instalador. Para CAs customizadas:

```ini
# Inventory — receptor com CA customizada
receptor_tls_cert=/path/to/receptor.crt
receptor_tls_key=/path/to/receptor.key
receptor_ca_cert=/path/to/receptor-ca.crt
```

### Gerar CA e certificados manualmente

```bash
# 1. Criar CA do mesh
receptor --cert-init commonname="mesh CA" bits=4096 \
  outcert=/etc/receptor/tls/ca/mesh-CA.crt \
  outkey=/etc/receptor/tls/ca/mesh-CA.key

# 2. Criar CSR para cada nó (usar FQDN como commonname e nodeid)
receptor --cert-makereq commonname=<FQDN> bits=4096 nodeid=<FQDN> \
  outreq=/etc/receptor/tls/<FQDN>.csr \
  outkey=/etc/receptor/tls/<FQDN>.key \
  ipaddress=<IP>

# 3. Assinar CSR com a CA
receptor --cert-signreq verify=yes \
  cacert=/etc/receptor/tls/ca/mesh-CA.crt \
  cakey=/etc/receptor/tls/ca/mesh-CA.key \
  req=/etc/receptor/tls/<FQDN>.csr \
  outcert=/etc/receptor/tls/<FQDN>.crt \
  notafter="2034-07-29T20:48:02Z"

# 4. Permissões corretas
chown -R receptor: /etc/receptor
chmod 0640 /etc/receptor/tls/<FQDN>.key
```

## Deprovisioning de Nós

```bash
# Remover nó individual usando o installer
cd /path/to/aap-installer/
./setup.sh -e '{"deprovision_node": "exec1.exemplo.org"}'

# Remover grupo completo
./setup.sh -e '{"deprovision_group": "instance_group_dmz"}'
```
