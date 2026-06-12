# Glossário — AAP 2.6 Automações

**ansible.platform collection**
Collection oficial Red Hat para Configuration as Code. Contém módulos para gerenciar todos os objetos do AAP via API (Controller, Hub, Gateway).

**ansible-builder**
Ferramenta CLI para construir imagens de Execution Environment. Lê um arquivo `execution-environment.yml` e gera um Containerfile para build com Podman/Docker.

**ansible-navigator**
Interface TUI/CLI para execução de playbooks dentro de Execution Environments localmente. Substitui o `ansible-playbook` para desenvolvimento com EEs.

**Rulebook**
Arquivo YAML que define um conjunto de regras para o EDA Controller. Cada regra tem: fonte de evento (source), condição (condition), e ação (action).

**Rulebook Activation**
Processo em execução no EDA Controller que monitora uma fonte de eventos com base em um Rulebook. Pode ter status: Running, Failed, Stopped.

**Decision Environment (DE)**
Imagem de contêiner para execução dos Rulebooks no EDA Controller. Análogo ao Execution Environment para o Automation Controller.

**Source Plugin**
Conector para fonte de eventos no EDA. Exemplos: `ansible.eda.webhook`, `ansible.eda.kafka`, `ansible.eda.alertmanager`.

**Condition**
Expressão Python no Rulebook que filtra eventos recebidos. Determina se a ação deve ser executada.

**Action**
O que o EDA faz quando uma condição é satisfeita: `run_job_template`, `run_workflow_template`, `run_playbook`, `debug`, `post_event`.

**Configuration as Code (CaC)**
Prática de definir e gerenciar a configuração do AAP via arquivos YAML versionados em Git, usando a `ansible.platform` collection.

**Execution Environment (EE)**
Imagem OCI que contém ansible-core + collections + dependências Python + dependências de sistema. Garante execução reprodutível e consistente de playbooks.

**execution-environment.yml**
Arquivo de definição para construir um Execution Environment com ansible-builder. Define imagem base, collections Galaxy, dependências Python e sistema.

**ansible-lint**
Ferramenta de análise estática de código Ansible. Verifica boas práticas, idempotência, formatação e segurança nos playbooks e roles.

**Survey**
Formulário configurável em um Job Template que solicita variáveis ao usuário antes da execução. Substitui o input manual de `extra_vars`.

**Idempotência**
Propriedade de uma task Ansible onde executar múltiplas vezes produz o mesmo resultado que executar uma vez. Usar módulos declarativos em vez de `shell`/`command`.

**Handler**
Task especial no Ansible que só executa quando notificada por outra task (`notify`). Usado para reiniciar serviços somente quando houve mudança.

**block/rescue/always**
Estrutura de tratamento de erros no Ansible. `block` contém as tasks principais, `rescue` executa em caso de falha (rollback), `always` sempre executa.
