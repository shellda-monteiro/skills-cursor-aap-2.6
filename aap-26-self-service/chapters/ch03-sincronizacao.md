# Ch03 — Sincronização de Job Templates com o AAP

## O que é Sincronizado

| Dado | Origem AAP | Como aparece no Portal |
|---|---|---|
| **Organizações** | Organizations | Contexto de acesso |
| **Usuários** | Users | Login via OAuth AAP |
| **Times** | Teams | Grupos de permissão |
| **Job Templates** | Job Templates | Self-service templates |

A sincronização roda de forma agendada (configurável via Helm values) e pode ser **disparada manualmente** por um administrador no portal.

## Configuração de Sincronização no Helm values

```yaml
catalog:
  providers:
    rhaap:
      orgs: "MinhaOrganizacao"    # apenas UMA organização

      sync:
        # Sincronização de usuários/times (obrigatório)
        orgsUsersTeams:
          schedule:
            frequency: { minutes: 60 }   # a cada 60 minutos
            timeout: { minutes: 15 }

        # Sincronização de Job Templates (opcional, padrão: true)
        jobTemplates:
          enabled: true
          schedule:
            frequency: { minutes: 60 }
            timeout: { minutes: 15 }

          # Filtros (opcional — sem filtro = sincroniza TODOS os templates)
          surveyEnabled: true    # apenas templates COM survey habilitado
          labels:                # apenas templates com estas labels (OR)
            - "self-service"
            - "usuario-final"
```

## Filtros de Job Template

### Por Label do AAP

```yaml
# Sincronizar APENAS templates com label "self-service" ou "portal"
jobTemplates:
  labels:
    - "self-service"
    - "portal"
```

> As labels são normalizadas: `CaC` → `cac`, `App-Dev` → `app-dev`

### Por Survey

```yaml
# Apenas templates COM survey (formulário) habilitado
jobTemplates:
  surveyEnabled: true

# Apenas templates SEM survey
jobTemplates:
  surveyEnabled: false

# Todos os templates (padrão quando omitido)
# jobTemplates:
#   surveyEnabled: (omitir)
```

### Combinando filtros (AND)

```yaml
# Templates que têm a label "self-service" E têm survey habilitado
jobTemplates:
  surveyEnabled: true
  labels:
    - "self-service"
```

## Disparar Sincronização Manual

```
Portal → (login como admin AAP) → Administration
  → Synchronize Ansible Automation Platform
  → Escolher: Identity (usuários/times) ou Templates
  → Confirm
```

## Como Preparar os Job Templates no AAP para o Portal

```
Para um template aparecer bem no portal:
1. Adicionar label "self-service" ao template (para usar filtro por label)
2. Configurar Survey com campos claros e descrições em português
3. Definir um ícone/descrição do template
4. Testar o formulário gerado no portal após a sincronização
```

## Atualizar Configuração (Helm Upgrade)

```bash
# Editar values.yaml (ex: adicionar nova label ao filtro)
helm upgrade self-service-portal \
  openshift-helm-charts/ansible-automation-platform-self-service \
  -f values.yaml \
  -n self-service-portal

# A sincronização rodará no próximo ciclo ou manualmente
```
