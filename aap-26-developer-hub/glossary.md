# Glossário — Ansible Plug-ins para Developer Hub

**Red Hat Developer Hub (RHDH)** — Plataforma IDP (Internal Developer Platform) baseada em Backstage que centraliza ferramentas, templates e documentação para desenvolvedores da organização.

**Backstage** — Framework open-source criado pelo Spotify para portais internos de desenvolvedor. O RHDH é a distribuição suportada pela Red Hat.

**Ansible Plug-ins** — Conjunto de plug-ins do RHDH que adicionam uma experiência Ansible-first: home page, learning paths, software templates, links para Dev Spaces e Automation Controller.

**Software Template** — Template no RHDH que cria automaticamente um repositório Git com estrutura padronizada de projeto Ansible (playbook ou collection) ao ser acionado pelo desenvolvedor.

**OpenShift Dev Spaces** — IDEs web sob demanda rodando no OpenShift. O desenvolvedor acessa um VS Code completo no browser, pré-configurado com as ferramentas Ansible necessárias.

**IDP (Internal Developer Platform)** — Portal interno que unifica ferramentas, documentação e workflows do desenvolvedor, reduzindo a fricção no ciclo de desenvolvimento.

**Dynamic Plugins** — Mecanismo do RHDH para adicionar funcionalidades sem recompilar a aplicação. Os Ansible Plug-ins são instalados como dynamic plugins.

**OCI artifact** — Formato de entrega dos plug-ins via container registry (registry.redhat.io). Método recomendado pois não requer downloads manuais.

**devfile.yaml** — Arquivo de configuração que define o ambiente de desenvolvimento no Dev Spaces (ferramentas, versões, comandos). Gerado automaticamente pelos software templates Ansible.

**.devfile.yaml** — Arquivo específico do projeto que configura o Dev Spaces: EE a ser usado, extensões do VS Code, variáveis de ambiente, comandos de build/test.
