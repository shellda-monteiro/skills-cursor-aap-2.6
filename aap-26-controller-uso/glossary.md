# Glossário — Automation Controller (Uso)

**Job Template** — Definição parametrizada de um job Ansible: playbook, inventory, credenciais, variáveis. Base da automação no Controller.

**Workflow Job Template** — Grafo de execução encadeando Job Templates, Project Syncs, Inventory Syncs e outros Workflows em nós com condições (success/failure/always).

**Prompt on Launch** — Flag em campos do Job Template que permite sobrescrever o valor no momento do launch via UI, API ou schedule.

**Survey** — Formulário gerado a partir de um Job Template para coletar variáveis do usuário antes da execução (sem expor o playbook).

**Job Slicing** — Distribuição do inventory em N fatias para execução paralela em múltiplos nós. Gera um Workflow Job com N sub-jobs.

**Instance Group** — Agrupamento lógico de nós de execução. Permite direcionar jobs para conjuntos específicos de nós (ex: produção, DMZ).

**Container Group** — Instance Group onde os jobs rodam como pods efêmeros no OpenShift ou Kubernetes.

**Constructed Inventory** — Inventory que filtra e agrupa hosts de outros inventories usando expressões Jinja2.

**Smart Inventory** — (DEPRECIADO) Inventory filtrado por fatos armazenados. Substituído por Constructed Inventory.

**OAuth 2 Token** — Token de acesso para autenticação programática na API. Configurável com escopo (read/write) e timeout de expiração.

**Webhook** — Mecanismo de integração onde um push em repositório GitHub/GitLab dispara automaticamente um Job Template ou Workflow.

**set_stats** — Módulo Ansible para publicar variáveis de um job para nós downstream em workflows.

**Convergence Node** — Nó em workflow que aguarda a conclusão de múltiplos nós upstream antes de executar.

**Approval Node** — Nó de workflow que pausa a execução aguardando aprovação humana com timeout configurável.

**Activity Stream** — Log de auditoria de todas as ações realizadas no Controller (criações, modificações, lançamentos).
