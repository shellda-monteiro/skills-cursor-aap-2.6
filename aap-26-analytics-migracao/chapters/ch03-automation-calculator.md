# Ch03 — Automation Calculator: ROI e Cálculo de Savings

## O que é o Automation Calculator

O Automation Calculator (disponível em console.redhat.com → Automation Analytics) fornece gráficos, métricas e cálculos para demonstrar o **valor financeiro e de tempo** do investimento em automação.

## Fórmulas de Cálculo

```
Custo Manual por Template =
  (tempo de execução manual em 1 host × total de hosts em todas as execuções) × custo por hora

Custo de Automação por Template =
  custo de automação por hora × total de horas gastas em execuções do template

Savings por Template = Custo Manual - Custo de Automação

Total Savings = Σ Savings de todos os templates
```

## Variáveis do Cálculo

| Variável | Descrição | Padrão |
|---|---|---|
| **Manual cost of automation** | Custo aproximado de um profissional executando a tarefa manualmente ($/hora) | Configurável |
| **Cost of automation** | Custo associado à execução do job template ($/hora) | Configurável |
| **Automation time** | Tempo de execução do job template | Medido pelo Controller |
| **Number of hosts** | Hosts no inventory onde o template executa | Medido pelo Controller |
| **Manual duration** | Tempo para executar a mesma tarefa manualmente por host | **Padrão: 60 minutos** |

> O cálculo inicial usa **valores padrão** — ajustar os valores reais aumenta a precisão.

## Top Templates

Listagem dos **25 templates mais executados** em todo o ambiente:
- Ordenados por contagem de execuções (decrescente)
- Para cada template: ajustar o campo "tempo manual" para o tempo real da tarefa manual equivalente
- Total Savings atualiza dinamicamente

### Detalhes por Template

- **Total elapsed sum:** tempo total de execução do template
- **Success elapsed sum:** tempo total em execuções bem-sucedidas
- **Failed elapsed sum:** tempo total em execuções com falha
- **Automation percentage:** percentual que este template representa no total de automação
- **Associated organizations:** organizações que usam o template
- **Associated clusters:** clusters onde o template executa

> Automação savings calculations **não são salvas** — são calculadas em tempo real.

## Como Apresentar ROI para o Cliente

### Cenário de Exemplo: Patching Mensal

```
Template: "Patching RHEL Mensal"
Execuções por mês: 1
Hosts por execução: 300

Configuração no Calculator:
  - Manual duration: 15 min/host (um sysadmin executando manualmente)
  - Manual cost: R$ 150/hora (custo médio do analista)
  - Automation cost: R$ 20/hora (custo do AAP prorated)

Cálculo:
  Custo manual = (15min/60 × 300 hosts) × R$150/hora = R$ 11.250/mês
  Custo automação ≈ R$ 2/execução
  Savings mensal ≈ R$ 11.248
  Savings anual ≈ R$ 134.976
```

## Dicas para Consultores

```
✓ Antes da apresentação: ajustar custo por hora para refletir a realidade do cliente
✓ Usar os "Top Templates" para focar no top 5 com maior impacto
✓ Habilitar "use_fact_cache=True" nos templates para coletar dados de hosts
✓ Exportar screenshot do Calculator para relatórios (valores não são salvos)
✓ Comparar savings mês a mês para demonstrar crescimento do programa
```
