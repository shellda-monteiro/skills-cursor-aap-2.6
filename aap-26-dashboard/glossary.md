# Glossário — Automation Dashboard

**Automation Dashboard** — Aplicação containerizada separada do AAP que consolida métricas de execução de até 3 instâncias AAP da mesma versão. Alternativa local ao Automation Analytics (console.redhat.com).

**clusters.yaml** — Arquivo de configuração que define as instâncias AAP conectadas ao dashboard, contendo tokens OAuth2, refresh tokens e schedules de sincronização.

**setclusters** — Comando de gerenciamento (`manage.py setclusters`) que carrega o `clusters.yaml` no banco de dados do dashboard.

**getclusters** — Comando de gerenciamento que retorna a configuração atual de clusters armazenada no banco, opcionalmente descriptografada com `--decrypt`.

**syncdata** — Comando que sincroniza dados de jobs e hosts das instâncias AAP conectadas para o banco do dashboard.

**Token Rotation** — Processo automático do dashboard de renovar o `access_token` usando o `refresh_token`. O `refresh_token` é de uso único; após rotação, apenas o token armazenado internamente é válido.

**scheduler_syncjob** — Tabela do banco de dados do dashboard que rastreia tarefas de sincronização. Jobs travados com status `pending/running` podem bloquear sincronizações futuras.

**Monthly AAP cost** — Custo mensal configurável no dashboard para cálculo de ROI; inclui licença, labor e infraestrutura do AAP, rateado por dia para os relatórios.

**bundle_install** — Flag do inventory que instrui a instalação a usar os containers do bundle local (offline) em vez de baixar do registry (padrão: `true`).
