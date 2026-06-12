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

## Intelligent Assistant (Chatbot AAP) — Deploy rápido no OCP

```yaml
# 1. Criar secret (namespace aap)
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-configuration-secret
  namespace: aap
stringData:
  chatbot_llm_provider_type: openai    # rhelai_vllm | rhoai_vllm | openai | azure_openai
  chatbot_url: https://api.openai.com/v1
  chatbot_model: gpt-4o-mini
  chatbot_token: <api_key>
  # Opcional: habilitar MCP (dados em tempo real do AAP)
  aap_gateway_url: http://myaap
  aap_controller_url: http://myaap-controller-service
```

```yaml
# 2. Referenciar no Operator AAP (spec.lightspeed)
spec:
  lightspeed:
    disabled: false
    chatbot_config_secret_name: chatbot-configuration-secret
```

```bash
# 3. Verificar pods e MCP containers
oc get pods -n aap | grep lightspeed
oc describe pod myaap-lightspeed-chatbot-api-* -n aap | grep -A5 "Containers:"
# ansible-mcp-lightspeed  → MCP Gateway ativo
# ansible-mcp-controller  → MCP Controller ativo
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
