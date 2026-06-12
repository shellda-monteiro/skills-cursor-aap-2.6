# Ch04 — Automation Analytics: Savings Planner, Reports e Job Explorer

## Acesso ao Automation Analytics

Analytics disponível em: **console.redhat.com → Automation Analytics**

> Requer que os controllers enviem dados para o Red Hat Hybrid Cloud Console (HCC). Ativar em: Controller Settings → Automation Analytics → Gather data for Automation Analytics.

---

## Automation Savings Planner

Permite **planejar e acompanhar** iniciativas de automação com projeção de savings ao longo do tempo.

### Criar um Savings Plan

```
Automation Analytics → Savings Planner → Add Plan

Passo 1 — Informações gerais:
  - Nome: "Patching RHEL - Satellite"
  - Descrição: "Automação do ciclo mensal de patching"
  - Tipo: Infrastructure
  - Nº de hosts: 300
  - Duração manual: 15 min/host
  - Frequência: Mensal

Passo 2 — Tarefas do plano:
  - [ ] Identificar hosts elegíveis no Satellite
  - [ ] Aplicar errata via playbook
  - [ ] Validar serviços pós-patching
  - [ ] Gerar relatório de conformidade

Passo 3 — Vincular ao Job Template:
  - Selecionar: "Patching RHEL Mensal" (Job Template do Controller)
  → Salvar
```

### Visualizar Savings ao Longo do Tempo

```
Savings Plan → nome do plano → aba Statistics
→ Gráfico: Monetary Savings ou Time Savings projetados por ano
→ Subtracts: automated cost - manual cost = resources saved
```

### Usar para Vendas/Consultoria

```
Sequência recomendada:
1. Criar savings plan por iniciativa de automação
2. Vincular ao job template real após implementação
3. Após algumas execuções: revisitar o plan → statistics mostram savings reais
4. Exportar como evidência para relatório de ROI
```

---

## Reports — Visão Geral das Automações

Navegação: **Automation Analytics → Reports**

| Report | O que mostra |
|---|---|
| **Automation Calculator** | ROI financeiro e de tempo |
| **Clusters** | Status e volume de execuções por cluster |
| **Job Explorer** | Detalhes de cada job em todos os clusters |
| **Savings Planner** | Planos de savings por iniciativa |
| **Hosts** | Hosts gerenciados por template |
| **Templates** | Frequência e status dos templates |
| **Organizations** | Uso de automação por organização |

---

## Job Explorer

Visão detalhada de **todos os jobs** rodados em todos os clusters/organizações.

### Filtros disponíveis

- Status (failed, successful, running, pending)
- Job type
- Cluster
- Organization
- Inventory
- Template

### Casos de uso

```
1. Investigar falhas recentes em produção:
   Status = Failed → Cluster = producao → últimas 24h
   → clicar no job → ir direto para detalhes no Controller

2. Auditoria de execuções:
   Organization = financeiro → Período = último trimestre
   → exportar para relatório de governança

3. Identificar templates mais lentos:
   ordenar por "elapsed time" (decrescente)
   → candidatos para otimização (forks, job slicing)

4. Filtrar apenas jobs de nível raiz (excluir sub-jobs de workflows):
   → filtro "Filter out nested workflows and jobs"
```

### Drill-down do gráfico de Clusters

```
Clusters → clicar em barra do gráfico → abre Job Explorer
→ já filtrado pelo cluster e data selecionados
→ adicionar filtro: Status = Failed
→ lista de jobs com falha naquele dia → link para detalhes no Controller
```

---

## Automation Analytics Data Dictionary

O Data Dictionary descreve **quais dados** o Controller envia para o HCC:

Dados enviados incluem:
- Job executions (template name, status, elapsed time, host count)
- Cluster information (version, license, instance count)
- Host data (when `use_fact_cache=True` is set)
- Organization and inventory metadata

> Referência completa: [Red Hat KCS - Ansible Automation Platform Data Dictionary](https://access.redhat.com/articles/)

### Configurar envio de dados

```
Controller → Settings → Automation Analytics
  → Gather data for Automation Analytics: ON
  → Insights for Ansible enabled: ON (requer Red Hat login)
```
