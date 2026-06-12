# Patterns — Migração e Analytics

## Pattern 1: Migração RPM → Containerizado (Produção)

**Cenário:** Cliente com AAP 2.6 RPM em 3 VMs quer migrar para containerizado (Podman) em novos servidores.

```
FASE 1 — Preparação (semana antes)
├── Verificar que o RPM está na última versão async de 2.6
├── Mapear topologia: quais VMs são Controller, Hub, Gateway
├── Levantar o que é fora de escopo (EDA? Instance Groups? EEs customizados?)
└── Preparar servidores RHEL alvo e instalar pré-requisitos

FASE 2 — Export (janela de manutenção)
├── Notificar usuários da janela de manutenção
├── Criar estrutura de artifact: mkdir -p artifact/{controller,gateway,hub}
├── pg_dump dos 3 bancos → .pgc files
├── Coletar SECRET_KEYs → secrets.yml
├── tar + sha256sum → artifact.tar
└── scp para nó gateway do ambiente alvo

FASE 3 — Import (mesma janela)
├── Instalar AAP containerizado no alvo (usando secrets.yml)
├── Verificar PostgreSQL 15 no alvo
├── Backup do alvo recém-instalado (ponto de rollback)
├── Parar serviços (exceto DB)
├── pg_restore dos 3 bancos
└── Reiniciar serviços

FASE 4 — Validação
├── Login na UI do novo ambiente
├── Verificar Job Templates, Inventories, Credentials
├── Executar job de smoke test
├── Recriar manualmente: Instance Groups, EDA config, EEs customizados
└── Redirecionar DNS/LB para o novo ambiente
```

---

## Pattern 2: Demonstração de ROI para Decisão de Projeto

**Cenário:** Preciso convencer o gestor a investir em AAP apresentando savings potenciais.

```
Passo 1 — Levantar dados das automações planejadas:
  Automation: "Patching RHEL via Satellite"
    → 400 servidores
    → Frequência: mensal
    → Tempo manual estimado: 20 min/servidor
    → Custo médio do analista: R$ 120/hora

Passo 2 — Calcular no papel:
  Manual = (20min/60 × 400 hosts) × R$ 120/hora = R$ 16.000/mês
  Automação ≈ R$ 50/mês (custo de infra prorated)
  Savings mensal = R$ 15.950 → Anual = R$ 191.400

Passo 3 — Criar Savings Plan no Analytics:
  Automation Analytics → Savings Planner → Add Plan
    Nome: "Patching RHEL - Satellite"
    Hosts: 400
    Manual duration: 20 min
    Frequência: Monthly
    Tipo: Infrastructure
  → Salvar como evidência

Passo 4 — Após implementação:
  → Vincular ao Job Template real
  → Após 3 meses: comparar projetado vs real no Statistics tab
  → Apresentar gráfico de savings acumulado ao longo de 12 meses
```

---

## Pattern 3: Auditoria de Falhas com Job Explorer

**Cenário:** Gerente de TI quer saber quais automações falharam no trimestre e por quê.

```
1. console.redhat.com → Automation Analytics → Job Explorer

2. Filtrar:
   - Status: Failed
   - Cluster: cluster-producao
   - Período: últimos 90 dias
   - Filter nested workflows: ON (ver apenas jobs raiz)

3. Ordenar por: Created (decrescente)

4. Para cada falha relevante:
   - Clicar no nome do job → abre detalhes no Controller
   - Verificar: template, inventory, hosts afetados, stderr

5. Agrupar por template → identificar templates com maior taxa de falha

6. Exportar dados → relatório trimestral de confiabilidade de automação
```

---

## Pattern 4: Monitoramento Contínuo de Savings

**Ciclo mensal para consultores:**

```
1. Início do mês: revisar Top Templates no Calculator
   → Ajustar "manual duration" para templates novos

2. Atualizar custos anuais no Calculator
   → Cost of automation (ajuste de licença/infra)
   → Manual cost (ajuste de salários)

3. Job Explorer: revisar top 10 templates por volume de execução
   → Identificar candidatos para otimização (slicing, forks)

4. Savings Planner: revisar planos em execução
   → Planos vinculados a templates reais → verificar Statistics
   → Novos projetos: criar savings plans antes de iniciar

5. Exportar screenshots → relatório mensal de programa de automação
```
