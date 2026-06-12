# Ch05 — Rede, Portas e Firewall

## Tabela Completa de Portas (Containerizado / RPM)

| Destino | Porta | Protocolo | Serviço | Obrigatório |
|---|---|---|---|---|
| Automation Hub | 22 | TCP/SSH | Gerenciamento (install/upgrade) | Sim |
| Automation Hub | 80/443 | TCP/HTTP(S) | Pull collections e EEs | Sim |
| Automation Controller | 22 | TCP/SSH | Gerenciamento | Sim |
| Automation Controller | 80/443 | TCP/HTTP(S) | API e UI (via Gateway) | Sim |
| Automation Controller | **27199** | TCP/Receptor | **Automation Mesh** | Sim |
| EDA Controller | 22 | TCP/SSH | Gerenciamento | Sim |
| EDA Controller | 80/443 | TCP/HTTP(S) | API e UI (via Gateway) | Sim |
| EDA Controller | 8443 | TCP/HTTPS | Recepção de event stream | Sim |
| Execution Node | 22 | TCP/SSH | Gerenciamento | Sim |
| Execution Node | **27199** | TCP/Receptor | **Mesh peering** | Sim |
| Hop Node | 22 | TCP/SSH | Gerenciamento | Sim |
| Hop Node | **27199** | TCP/Receptor | **Relay Receptor** | Sim |
| PostgreSQL | 22 | TCP/SSH | Gerenciamento | Sim |
| PostgreSQL | **5432** | TCP | **Banco de dados** | Sim |
| Redis | **6379** | TCP | Cache e filas | Sim |
| Redis Cluster Bus | **16379** | TCP | Cluster Redis HA | Só em cluster |
| Platform Gateway | 80/443 | TCP/HTTP(S) | Interface principal | Sim |
| Platform Gateway | 8443 | TCP/HTTPS | nginx interno | Sim |
| Mesh Ingress (OCP) | 443 | TCP/HTTPS | Execution nodes → OCP route | Se mesh ingress |

## URLs Externas Necessárias (Firewall de Saída)

### Red Hat Registry e Subscriptions
```
https://api.access.redhat.com:443        # Account services, subscriptions
https://cert-api.access.redhat.com:443   # Insights data upload
https://cert.console.redhat.com:443      # Inventory upload, Cloud Connector
https://console.redhat.com:443           # Insights dashboard
https://sso.redhat.com:443               # SSO
```

### Automation Hub (Collections e EEs)
```
https://console.redhat.com:443
https://catalog.redhat.com:443
https://automation-hub-prd.s3.amazonaws.com        # Collections S3
https://automation-hub-prd.s3.us-east-2.amazonaws.com
https://galaxy.ansible.com:443                     # Community content
https://ansible-galaxy-ng.s3.dualstack.us-east-1.amazonaws.com
https://registry.redhat.io:443                     # Container images (Red Hat)
https://cert.console.redhat.com:443                # Certified collections
```

### Container Images (quay.io CDN — CRÍTICO desde abril 2025)
```
cdn.quay.io:443
cdn01.quay.io:443
cdn02.quay.io:443
cdn03.quay.io:443
cdn04.quay.io:443    # NOVO — adicionado abril 2025
cdn05.quay.io:443    # NOVO
cdn06.quay.io:443    # NOVO
```

**ATENÇÃO:** Desde 01/04/2025, novos endpoints foram adicionados ao quay.io CDN. Regras de firewall que bloqueiam por IP vão falhar — usar **hostnames** nas regras.

## Regras Resumidas por Função

### Para instalar (installer node precisa acessar)
```
todos os nós alvos : 22/TCP (SSH)
registry.redhat.io  : 443/TCP (imagens)
cdn*.quay.io        : 443/TCP (imagens)
```

### Para operar (inter-componentes)
```
Gateway  → Controller   : 80/443 ou 8443
Gateway  → Hub          : 80/443 ou 8444
Gateway  → EDA          : 80/443 ou 8445
Controller → DB         : 5432
Controller → Redis      : 6379
Controller → Exec nodes : 27199 (Receptor)
EDA      → Redis        : 6379
Hub      → DB           : 5432
Hub      → Redis        : 6379
Hub      → EDA          : 6379
```

### Para webhooks EDA (entrada)
```
Fonte externa → EDA : 443 ou porta customizada
(Configurar no inventory: eda_nginx_disable_https=false)
```

## Configuração de TLS

### AAP gera certificados (padrão)
- Certificados auto-assinados gerados durante instalação
- Aceitáveis para testes; **não usar em produção** sem CA customizada

### CA Customizada (recomendado)
```ini
[all:vars]
# Fornecer CA para o AAP assinar todos os certificados internos
ca_tls_cert=/path/to/ca.crt
ca_tls_key=/path/to/ca.key
```

### Certificados individuais
```ini
# Cada serviço recebe seu próprio certificado
gateway_tls_cert=/path/to/gw.crt
gateway_tls_key=/path/to/gw.key
# (idem para controller, hub, eda)
```

### Certificado da CA Raiz Institucional
```ini
# Adicionar CA root ao trust store do AAP
custom_ca_cert=/path/to/corporate-root-ca.crt
```

## Portas do Receptor (Automation Mesh)

```ini
[execution_nodes]
exec1.example.org receptor_listener_port=27199 receptor_peers=['controller.example.org']

[hop_nodes]
hop1.example.org receptor_listener_port=27199 receptor_peers=['controller.example.org']
```

- Usar hostnames consistentes em todos os nós — não misturar formatos
- Não usar `localhost` como nome de nó em instalações cluster

## Verificação de Conectividade Pós-Instalação

```bash
# Testar porta Receptor de um execution node
nc -zv controller.example.org 27199

# Testar PostgreSQL
nc -zv db.example.org 5432

# Testar Redis
nc -zv redis.example.org 6379

# Testar acesso ao registry (deve retornar 200)
curl -I https://registry.redhat.io/v2/

# Verificar quay.io CDN
curl -I https://cdn.quay.io/v2/
```
