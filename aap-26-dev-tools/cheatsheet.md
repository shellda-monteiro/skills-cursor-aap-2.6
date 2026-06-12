# Cheatsheet — Dev Tools (Navigator + Lightspeed)

## ansible-navigator — Comandos essenciais

```bash
# Executar playbook em EE
ansible-navigator run site.yml -i inventory/ -m stdout

# EE específico
ansible-navigator run site.yml \
  --eei registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest \
  -m stdout

# Explorar coleções do EE ativo
ansible-navigator collections

# Documentação de módulo
ansible-navigator doc ansible.builtin.template

# Explorar inventory
ansible-navigator inventory -i inventory/

# Replay de run anterior
ansible-navigator replay /tmp/artifacts/run.json

# Explorar EEs disponíveis
ansible-navigator images
```

## ansible-navigator.yml — Configuração mínima de projeto

```yaml
---
ansible-navigator:
  execution-environment:
    image: registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest
    pull:
      policy: missing
  mode: stdout
  playbook-artifact:
    enable: true
    save-as: /tmp/artifacts/{playbook_name}-{ts_utc}.json
  ansible:
    inventories:
      - ./inventory
```

## Lightspeed — Ativar no VS Code

```
1. Instalar extensão Ansible (Red Hat) no VS Code
2. Settings → Ansible → Lightspeed: Enabled ✓
3. Ctrl+Shift+P → Ansible: Connect to Lightspeed
4. Login Red Hat → autorizar
5. Escrever nome da task → Tab para aceitar sugestão
```

## Lightspeed — Trial gratuito

```
Trial 90 dias = apenas subscription AAP (sem IBM watsonx obrigatório)
→ Recursos: single-task, multi-task, playbook generation, explanations
→ Após trial: configurar cloud service ou on-premise para continuar
```

## Navigator — TUI atalhos

| Tecla | Ação |
|---|---|
| `Enter` | Selecionar/entrar |
| `Esc` | Voltar |
| `:<número>` | Ir para item/linha |
| `:collections` | Listar coleções |
| `:images` | Listar EEs |
| `:help` | Ajuda |
