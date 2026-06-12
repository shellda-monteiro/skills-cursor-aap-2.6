# Glossário — Migração e Analytics

**Migration Artifact** — Pacote tar contendo os dumps de banco (Controller, Gateway, Hub) e o arquivo `secrets.yml` com as chaves de criptografia, usado para transferir um ambiente AAP entre deployment types.

**RPM-based deployment** — Instalação do AAP via pacotes RPM em servidores RHEL. Modelo legado sendo substituído pelo containerizado.

**Container-based deployment** — Instalação do AAP em RHEL usando containers Podman (sem OpenShift). Atual recomendação para ambientes on-premises não-OCP.

**Managed AAP** — Serviço AAP gerenciado pela Red Hat no Red Hat Hybrid Cloud Console.

**secrets.yml** — Arquivo crítico do migration artifact contendo as `SECRET_KEY` de cada componente AAP e a chave de criptografia dos campos do banco do Hub.

**pg_dump / pg_restore** — Utilitários PostgreSQL usados para exportar e importar os bancos de dados durante a migração.

**Automation Analytics** — Serviço no console.redhat.com que agrega dados de execução dos Controllers registrados, fornecendo relatórios de ROI, savings e análise de jobs.

**Automation Calculator** — Ferramenta dentro do Analytics que calcula o retorno financeiro da automação comparando o custo manual versus automatizado.

**Automation Savings Planner** — Ferramenta para documentar iniciativas de automação, estimar savings futuros e vincular aos job templates reais para medir resultados.

**Job Explorer** — Interface analítica que exibe detalhes de todos os jobs executados em todos os clusters AAP registrados, com filtros por status, organização, template e cluster.

**use_fact_cache** — Flag em Job Templates que instrui o Controller a armazenar os facts do Ansible no banco, habilitando inventories inteligentes e enriquecendo os dados enviados ao Analytics.

**Red Hat Hybrid Cloud Console (HCC)** — Plataforma de console.redhat.com onde o Automation Analytics é hospedado e onde os dados de telemetria do Controller são enviados.
