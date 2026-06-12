# Ch04 — Ansible Lightspeed: Ativar, Usar e Administrar

## Ativar o Lightspeed no VS Code

### Pré-requisitos

1. VS Code instalado
2. Extensão **Ansible** do VS Code (Red Hat) instalada
3. Subscription AAP ativa (trial ou paga)
4. Conta Red Hat (para autenticação)

### Configurar a extensão Ansible no VS Code

```
VS Code → Extensions → Ansible (Red Hat) → Install

Settings → Ansible:
  → Lightspeed: Enabled: ✓
  → Lightspeed: URL: https://c.ai.ansible.redhat.com  (cloud service)
                     ou https://seu-servidor:8443       (on-premise)
  → Suggest: Enabled: ✓
```

### Autenticar

```
Command Palette (Ctrl+Shift+P) → Ansible: Connect to Lightspeed
→ Redireciona para login Red Hat no browser
→ Autorizar → retorna ao VS Code autenticado
```

## Gerar Recomendações de Tasks

```yaml
# 1. Criar arquivo .yml (playbook ou role task file)
# 2. Escrever o nome da task como comentário ou string:

- name: Install postgresql-server and run postgresql-setup  # ← prompt em inglês
  # → Lightspeed sugere o código abaixo automaticamente:
  ansible.builtin.package:
    name: postgresql-server
    state: present

- name: Initialize postgresql database
  ansible.builtin.command:
    cmd: postgresql-setup --initdb
    creates: /var/lib/pgsql/data/PG_VERSION
```

**Como aceitar:** `Tab` para aceitar sugestão, `Esc` para rejeitar.

## Geração de Playbook Completo

```
VS Code → Command Palette → Ansible: Lightspeed Playbook Generation

Prompt: "Install nginx on RHEL, configure a virtual host for app.empresa.org 
         on port 80, enable and start the service"

→ Lightspeed gera o playbook completo com:
   - tasks de instalação
   - template de configuração
   - handlers
   - meta informações
```

## Explicação de Playbook/Task

```
VS Code:
  1. Abrir playbook existente
  2. Selecionar o playbook ou uma task específica
  3. Command Palette → Ansible: Explain Playbook / Explain Task
  → Painel lateral exibe explicação em linguagem natural
```

**Uso em consultoria:** explicar playbooks legados para o time do cliente, documentar automações existentes.

## Administração do Lightspeed (para org admins)

```
Portal Lightspeed: https://c.ai.ansible.redhat.com/admin

→ Dashboard: usuários ativos, recomendações geradas, telemetria
→ Gerenciar usuários: habilitar/desabilitar por usuário
→ Configurar fine-tuning: vincular repositórios Git da organização
→ Desabilitar telemetria: opção por organização
```

## Ansible Code Bot

Bot que analisa repositórios GitHub/GitLab e **sugere atualizações automáticas** via Pull Request:

```
Casos de uso do Code Bot:
  - Módulos deprecados → sugerir equivalentes modernos
  - Sintaxe antiga → atualizar para formato atual
  - FQCNs ausentes → adicionar Fully Qualified Collection Names
  - Playbooks compatíveis com múltiplas versões do Ansible
```

**Instalação:**
```
GitHub: Settings → Apps → Ansible Code Bot → Install
GitLab: Integrations → Ansible Code Bot → Configure webhook
```

## Trial vs. Subscrição — O que está disponível

| Recurso | Trial 90 dias | Subscrição paga |
|---|---|---|
| Single-task recommendation | ✓ | ✓ |
| Multi-task recommendation | ✓ | ✓ |
| Playbook generation | ✓ | ✓ |
| Playbook explanation | ✓ | ✓ |
| Fine-tuning (modelos privados) | ✗ | ✓ |
| On-premise deployment | ✗ | ✓ |
| Ansible Code Bot | ✗ | ✓ |
| Admin dashboard | Limitado | ✓ completo |
