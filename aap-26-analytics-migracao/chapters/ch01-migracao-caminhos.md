# Ch01 — Migração: Caminhos Suportados e Pré-requisitos

## Regra Fundamental

> Só é possível migrar entre **deployment types da mesma versão**.  
> Migrar de RPM 2.6 → Containerizado 2.6 ✓  
> Migrar de RPM 2.4 → Containerizado 2.6 ✗ (requer upgrade para 2.6 primeiro)

## Caminhos de Migração Suportados no AAP 2.6

| Ambiente Fonte | Ambiente Alvo |
|---|---|
| RPM-based | **Container-based** |
| RPM-based | **OpenShift Container Platform (OCP)** |
| RPM-based | Managed AAP |
| Container-based | **OpenShift Container Platform (OCP)** |
| Container-based | Managed AAP |

> Migrações não listadas **não são suportadas** nesta versão.

## O que Está Fora do Escopo da Migração

Os seguintes itens **não são migrados automaticamente** — devem ser recriados manualmente no ambiente alvo:

| Componente | Ação necessária |
|---|---|
| **Event-Driven Ansible (EDA)** | Recriar configurações e rulebooks manualmente |
| **Instance Groups** | Recriar após a migração |
| **Conteúdo do Automation Hub** | Reimportar ou reconfigurar coleções/EEs |
| **CA customizado para Receptor Mesh** | Reconfigurar certificados da mesh |
| **Ambientes desconectados (air-gap)** | Não coberto pelo processo de migração |
| **Execution Environments customizados** | Reconstruir ou reimportar (apenas o EE padrão é migrado) |

## Pré-requisitos por Caminho de Migração

### RPM → Container-based

- Fonte: deployment RPM na **última versão async** da release atual
- Alvo: ambiente RHEL preparado para instalação containerizada
- Installer containerizado baixado para a mesma versão do fonte
- Espaço suficiente para dumps de banco e backups
- Conectividade de rede entre fonte e alvo

### RPM → OpenShift

- Mesmos pré-requisitos que RPM → Container
- Cluster OpenShift disponível (4.x)
- Operator AAP instalado no OCP
- Acesso ao namespace do OCP

### Container → OpenShift

- Mesmos pré-requisitos do caminho acima

## Fluxo Geral de Migração

```
1. Preparar e avaliar o ambiente FONTE
2. Exportar o ambiente fonte (databases + secrets)
3. Criar e verificar o migration artifact (tar + checksum)
4. Preparar e avaliar o ambiente ALVO
5. Importar o conteúdo migrado no alvo
6. Reconciliar o ambiente alvo pós-import
7. Validar o ambiente alvo
```

## Componentes Migrados

- Banco de dados do **Automation Controller** (`controller.pgc`)
- Banco de dados do **Platform Gateway** (`gateway.pgc`)
- Banco de dados do **Automation Hub** (`hub.pgc`)
- Arquivo `secrets.yml` com `SECRET_KEY` de cada componente e `hub_db_fields_encryption_key`
