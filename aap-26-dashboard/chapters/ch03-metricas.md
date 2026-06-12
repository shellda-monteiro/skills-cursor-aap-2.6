# Ch03 — Métricas e Relatórios

## Métricas Disponíveis no Dashboard

| Métrica | Descrição |
|---|---|
| **Successful jobs** | Jobs finalizados sem erro no período — indica saúde e confiabilidade |
| **Failed jobs** | Jobs com falha — revisar playbooks, credenciais ou inventory |
| **Hosts automated** | Hosts que executaram ao menos 1 job — útil para planejamento de licença/capacidade |
| **Hours of automation** | Soma de todos os tempos de execução — reflete carga de trabalho total |
| **Job run count** | Total de execuções independente de sucesso/falha — acompanhar volume e adoção |
| **Top 5 projects** | Projetos com maior volume de jobs — identifica os de maior atividade |
| **Top 5 users** | Usuários com maior uso de automação |

## Filtros para Relatórios

```
Dashboard → Reports → Filters:
  - Template: um ou mais Job Templates
  - Organization: uma ou mais organizações
  - Project: um ou mais projetos
  - Label: filtrar por labels atribuídas nos templates
  
  + Time period: período de análise (mais curto = análise específica; mais longo = tendências)
  + Currency: selecionar moeda (NÃO converte valores — ajustar manualmente os custos)
```

> **Labels:** precisam ser configuradas nos Job Templates do AAP para aparecerem no filtro.

## Cálculo de Savings (ROI)

```
Dashboard → Cost and Savings Analysis:

1. Monthly AAP cost: custo mensal do AAP (licença + labor + infra)
   → O dashboard prorratea por dia para os relatórios

2. Hourly rate for manually running the job: custo/hora do profissional executando manualmente

3. [Opcional] Time taken to create automation: tempo gasto escrevendo playbooks
   → Incluído no custo quando habilitado
```

**Cálculos resultantes:**

| Campo | Fórmula |
|---|---|
| Cost of manual automation | tempo_manual × hosts × custo_hora |
| Cost of automated execution | custo_mensal_AAP ÷ dias × dias_período |
| **Total savings** | manual - automated |
| Total hours saved | horas_manual - horas_execução |

## Salvar e Exportar Relatórios

```
Dashboard → relatório configurado → Save as Report
→ nome do relatório → salvar
→ recuperar depois em: Select a Report

Exportar:
→ Export as CSV: planilha com dados brutos (para stakeholders técnicos)
→ Save as PDF:  relatório formatado (para apresentações)
```

> Antes de exportar: configurar **Monthly AAP cost** e **Hourly rate** para garantir dados precisos de ROI no PDF.

## Upgrade do Dashboard

```bash
# 1. Baixar novo bundle
# 2. Extrair em NOVO diretório
# 3. Copiar inventory do diretório anterior

# 4. Adicionar seção [redis] ao inventory (obrigatório no upgrade)
# [redis]
# host.example.com ansible_connection=local
# [all:vars]
# redis_mode=standalone

# 5. Instalar
ansible-galaxy collection install -r requirements.yml
ansible-playbook -i inventory \
  ansible.containerized_installer.dashboard_install \
  --ask-become-pass

# 6. Atualizar clusters.yaml com refresh_token, client_id, client_secret
# 7. Recarregar clusters.yaml no container
podman cp clusters.yaml automation-dashboard-web:/tmp/
podman exec -it automation-dashboard-web \
  /venv/bin/python manage.py setclusters /tmp/clusters.yaml
```

## Desinstalar

```bash
ansible-playbook -i inventory \
  ansible.containerized_installer.dashboard_uninstall
```
