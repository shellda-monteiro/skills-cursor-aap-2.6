# Cheatsheet — Migração e Analytics

## Migração — Caminhos Suportados

```
RPM    → Container  ✓
RPM    → OCP        ✓
RPM    → Managed    ✓
Container → OCP     ✓
Container → Managed ✓

REGRA: mesma versão AAP (ex: 2.6 → 2.6)
FORA DO ESCOPO: EDA, Instance Groups, Hub content, EEs customizados, CA mesh
```

## Migração — Sequência

```bash
# 1. Dump dos bancos (fonte)
pg_dump -Fc -f controller/controller.pgc
pg_dump -Fc -f gateway/gateway.pgc
pg_dump -Fc -f hub/hub.pgc

# 2. Coletar SECRET_KEYs em secrets.yml

# 3. Criar artifact
tar cf artifact.tar artifact
sha256sum artifact.tar > artifact.tar.sha256

# 4. No alvo: instalar AAP (mesma versão) com secrets.yml
ansible-playbook ... -e @secrets.yml

# 5. Parar serviços (exceto DB), restaurar dumps
pg_restore -Fc controller/controller.pgc
pg_restore -Fc gateway/gateway.pgc
pg_restore -Fc hub/hub.pgc

# 6. Reiniciar serviços → validar
```

## Automation Calculator — Fórmulas

```
Manual cost/template = (tempo_manual × total_hosts) × $/hora
Automation cost/template = $/hora × horas_execucao
Savings = Σ (manual_cost - automation_cost)

Padrão manual duration: 60 min/host (ajustar!)
```

## Savings Planner

```
console.redhat.com → Automation Analytics → Savings Planner
→ Add Plan → definir hosts, duração manual, frequência
→ vincular ao Job Template → acompanhar savings reais
```

## Job Explorer — Filtros Úteis

```
Status=failed + Cluster=prod     → investigar falhas em produção
Organization=X + Período=trimestre → auditoria por organização
Ordenar por elapsed time         → templates lentos (candidatos a otimização)
Filtrar nested workflows         → ver apenas jobs raiz
```
