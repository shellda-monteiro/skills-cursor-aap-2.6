# Glossário — Dev Tools

**ansible-navigator** — Ferramenta CLI/TUI para desenvolver e executar playbooks Ansible dentro de Execution Environments. Substituto moderno do `ansible-playbook` direto.

**Ansible Development Tools (ADT)** — Pacote de ferramentas Red Hat para desenvolvimento de conteúdo Ansible, inclui `ansible-navigator`, `ansible-lint`, `molecule` e outros.

**TUI (Text-based User Interface)** — Modo interativo do ansible-navigator com navegação por menus e inspeção de resultados. Alternativa visual ao modo stdout.

**stdout mode** — Modo do ansible-navigator que reproduz a saída familiar do `ansible-playbook`, adequado para scripts e CI/CD.

**Playbook artifact** — Arquivo JSON gerado pelo navigator após cada execução, contendo tasks, output por host e facts. Permite replay e debugging colaborativo.

**ansible-navigator.yml** — Arquivo de configuração do navigator por projeto ou por usuário. Define EE padrão, modo, inventory, configurações de artefato, etc.

**Red Hat Ansible Lightspeed** — Serviço de IA generativa integrado ao VS Code que gera recomendações de código Ansible a partir de prompts em linguagem natural.

**IBM watsonx Code Assistant** — Serviço IBM de modelos de linguagem para código, base técnica do Lightspeed. Disponível como cloud service ou on-premise (Cloud Pak for Data).

**Fine-tuning** — Processo de treinar o modelo IBM watsonx com o conteúdo Ansible específico da organização (repositórios Git privados), tornando as sugestões mais precisas.

**Ansible Code Bot** — Bot para GitHub/GitLab que analisa repositórios e abre Pull Requests automaticamente sugerindo atualizações (módulos deprecados, FQCNs, etc.).

**Single-task recommendation** — Sugestão de código para uma única task Ansible baseada no nome/comentário da task.

**Multi-task recommendation** — Sugestão de código para múltiplas tasks de uma vez baseada em um prompt descritivo.

**FQCN (Fully Qualified Collection Name)** — Formato completo de referência a módulos Ansible: `namespace.collection.module` (ex: `ansible.builtin.template`). Recomendado para evitar ambiguidade.
