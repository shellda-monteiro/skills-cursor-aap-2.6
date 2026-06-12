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

**Automation Intelligent Assistant** — Interface de chat de IA embutida na UI do AAP que responde perguntas sobre o ambiente (jobs, inventories, templates) em linguagem natural usando LLMs. Componente do Lightspeed distinto do Coding Assistant.

**MCP (Model Context Protocol)** — Protocolo aberto que padroniza como aplicações fornecem contexto em tempo real a LLMs. No AAP, permite que o chatbot inteligente busque dados reais do Controller e Gateway.

**ansible-mcp-lightspeed** — Container MCP que expõe dados do Platform Gateway ao chatbot. Ativado ao configurar `aap_gateway_url` no chatbot-configuration-secret.

**ansible-mcp-controller** — Container MCP que expõe dados do Automation Controller (Job Templates, jobs, inventories) ao chatbot. Ativado ao configurar `aap_controller_url`.

**chatbot-configuration-secret** — Secret Kubernetes no namespace `aap` que contém as configurações do chatbot: modelo LLM, URL de inferência, token e variáveis MCP opcionais.

**vLLM** — Framework de serving de LLMs de alta performance suportado pelo RHEL AI, OpenShift AI e Red Hat AI Inference Server. Requerido para self-hosting de LLMs com o Intelligent Assistant.

**Tool calling** — Capacidade que um LLM precisa ter habilitada para interagir com serviços externos via MCP. Sem tool calling, o MCP server não funciona com o chatbot.

