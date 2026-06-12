# Glossário — AAP 2.6

## Componentes e Serviços

**Automation Controller**
Framework de orquestração de automação. Executa Job Templates, gerencia Inventários, Credenciais, Workflows e Schedules. Backend do AWX open-source.

**Automation Hub (Private)**
Repositório privado de Execution Environments (registry OCI) e Ansible Collections (backend Pulp). Substitui o Galaxy community para uso corporativo.

**EDA Controller (Event-Driven Ansible Controller)**
Interface para automação reativa baseada em Rulebooks. Processa eventos de múltiplas fontes e dispara Job Templates automaticamente.

**Execution Environment (EE)**
Imagem de contêiner OCI contendo ansible-core, collections e dependências Python necessárias para execução de playbooks. Substitui os Virtual Environments (vEnv) do AAP 1.x/AWX.

**Platform Gateway**
Serviço de autenticação e autorização central do AAP 2.6. Proxy reverso que unifica o acesso a todos os componentes em um único endpoint. Armazena sessões e JWTs no Redis centralizado.

**Automation Mesh**
Overlay network baseada no protocolo Receptor (TCP 27199) para distribuição de execução de playbooks. Permite execution nodes em zonas remotas sem VPN centralizada.

**Receptor**
Protocolo de comunicação do Automation Mesh (porta 27199 TCP). Suporta comunicação bidirecional, multi-hop e FIPS-compliant.

**ansible-builder**
Ferramenta CLI para construir imagens de Execution Environments customizadas. Define dependências via `execution-environment.yml`.

**ansible-navigator**
Interface de linha de comando (TUI) para desenvolvimento local dentro de Execution Environments.

## Topologias

**Growth Topology**
Topologia all-in-one para organizações iniciando com AAP. Todos os componentes em uma única VM. Sem redundância.

**Enterprise Topology**
Topologia de produção com componentes separados em hosts distintos, Redis cluster (6 nós), e capacidade de escalabilidade horizontal.

**Container Topology**
Deploy do AAP usando Podman em hosts RHEL. Instalação gerenciada pelo cliente via inventory Ansible.

**Operator Topology**
Deploy do AAP via Operator no OpenShift. Lifecycle gerenciado pelo Operator Hub; requer StorageClass RWX para Automation Hub.

## Infraestrutura

**StorageClass RWX**
StorageClass do tipo ReadWriteMany no OpenShift, necessária para o Automation Hub compartilhar storage entre múltiplos pods.

**IOPS (Input/Output Operations Per Second)**
Métrica de desempenho de storage. O AAP requer mínimo de 3000 IOPS para o volume do PostgreSQL. Abaixo disso → lentidão severa de banco.

**hstore**
Extensão PostgreSQL que armazena pares key-value em uma coluna. Obrigatória no banco de dados do Automation Hub.

**ICU (International Components for Unicode)**
Biblioteca necessária no PostgreSQL externo para suporte a collation. Obrigatória nas versões 15, 16 e 17.

**NTP (Network Time Protocol)**
Protocolo de sincronização de tempo. Obrigatório em todos os nós do AAP para evitar falhas de autenticação JWT e validação de certificados.

## Autenticação e Segurança

**RBAC (Role-Based Access Control)**
Controle de acesso baseado em papéis. No AAP: Organizations, Teams, Users e Roles definem permissões por componente.

**SAML (Security Assertion Markup Language)**
Protocolo de federação de identidade. Suportado pelo Platform Gateway para SSO corporativo.

**JWT (JSON Web Token)**
Token de autenticação stateless usado pelo Platform Gateway. Armazenado no Redis centralizado. Exige NTP sincronizado.

**Ansible Vault**
Ferramenta de criptografia para arquivos de inventory e variáveis sensíveis. Usar para proteger senhas no inventory file.

**mTLS (Mutual TLS)**
Autenticação mútua via certificados TLS. Suportado para comunicação AAP ↔ PostgreSQL externo e EDA event streams.

## Automação

**CaC (Configuration as Code)**
Prática de gerenciar configurações do AAP (Job Templates, Credenciais, Inventários) via código em repositório Git, usando o AWX Collection.

**Rulebook**
Arquivo YAML do EDA que define fontes de eventos, condições de filtragem e ações a executar (disparar Job Template, etc.).

**Decision Environment**
Imagem de contêiner para execução de Rulebooks no EDA Controller. Análogo ao Execution Environment para o Controller.

**Job Template**
Definição reutilizável de execução de playbook: inclui playbook, inventário, credenciais, variáveis extras e configurações de execução.

**Execution Node**
Nó do Automation Mesh responsável pela execução efetiva de playbooks. Pode estar em DMZ remota.

**Hop Node**
Nó relay do Automation Mesh. Repassa tráfego Receptor sem executar jobs. Baixo consumo de CPU/RAM.

**Hybrid Node**
Nó que combina funções de control + execution. Padrão em topologias growth.
