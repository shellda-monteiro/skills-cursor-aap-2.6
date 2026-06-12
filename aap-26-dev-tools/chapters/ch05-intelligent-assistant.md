# Ch05 — Automation Intelligent Assistant e Integração MCP

## O que é o Automation Intelligent Assistant

**Interface de chat de IA embutida diretamente na UI do AAP** que responde perguntas sobre o ambiente em linguagem natural. É o componente de chatbot do Red Hat Ansible Lightspeed — diferente do Coding Assistant (que gera código no VS Code).

```
AAP UI → ícone de chat (canto superior direito da taskbar)
  → "Quantos jobs falharam hoje na organização de Infra?"
  → "Quais Job Templates existem para patching?"
  → "Qual o status do inventory dinâmico do VMware?"
```

## Dois Componentes do Lightspeed

| Componente | Onde funciona | O que faz |
|---|---|---|
| **Coding Assistant** | VS Code (extensão Ansible) | Gera tasks/playbooks a partir de prompts |
| **Intelligent Assistant** | UI do AAP (chatbot embutido) | Responde perguntas sobre o ambiente AAP |

## Integração com MCP Server

O chatbot se conecta a **MCP servers** para buscar dados em tempo real do AAP ao responder perguntas:

| Container MCP | Variável que ativa | O que expõe |
|---|---|---|
| `ansible-mcp-lightspeed` | `aap_gateway_url` | Dados do Platform Gateway (usuários, orgs, tokens) |
| `ansible-mcp-controller` | `aap_gateway_url` + `aap_controller_url` | Job Templates, jobs, inventories, credentials do Controller |

> **Status no AAP 2.6:**
> - OCP (Operator): **Generally Available**
> - Containerizado (RHEL): **Technology Preview**

## Provedores LLM Suportados

| Provedor | Tipo | `chatbot_llm_provider_type` |
|---|---|---|
| Red Hat Enterprise Linux AI (vLLM) | On-premise (self-hosted) | `rhelai_vllm` |
| Red Hat OpenShift AI (vLLM) | On-premise (self-hosted) | `rhoai_vllm` |
| Red Hat AI Inference Server | On-premise (self-hosted) | — |
| OpenAI | Cloud (SaaS) | `openai` |
| Microsoft Azure OpenAI | Cloud (SaaS) | `azure_openai` |

> O LLM **deve ter tool calling habilitado** para o MCP funcionar (interação com serviços da plataforma).

## Deploy no OpenShift (Operator)

### Passo 1: Criar o Secret de Configuração

```yaml
# Exemplo com OpenShift AI (modelo Red Hat Granite)
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-configuration-secret
  namespace: aap
stringData:
  chatbot_llm_provider_type: rhoai_vllm
  chatbot_url: https://llm.apps.mycluster.empresa.org/v1
  chatbot_model: granite-3.3-8b-instruct
  chatbot_token: <token_do_modelo>
```

```yaml
# Exemplo com OpenAI
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-configuration-secret
  namespace: aap
stringData:
  chatbot_llm_provider_type: openai
  chatbot_url: https://api.openai.com/v1
  chatbot_model: gpt-4o-mini
  chatbot_token: <openai_api_key>
```

```yaml
# Exemplo com Azure OpenAI
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-configuration-secret
  namespace: aap
stringData:
  chatbot_llm_provider_type: azure_openai
  chatbot_url: https://meurecurso.openai.azure.com
  chatbot_model: gpt-4o-mini
  chatbot_token: <azure_api_key>
  chatbot_model_config_extras: '{"api_version": "2025-01-01-preview"}'
```

### Com MCP server habilitado (acesso em tempo real ao AAP)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-configuration-secret
  namespace: aap
stringData:
  chatbot_llm_provider_type: openai
  chatbot_url: https://api.openai.com/v1
  chatbot_model: gpt-4o
  chatbot_token: <api_key>
  # MCP: expor dados do Gateway + Controller para o chatbot
  aap_gateway_url: http://myaap                       # URL interna do Gateway no OCP
  aap_controller_url: http://myaap-controller-service  # URL interna do Controller no OCP
```

### Passo 2: Habilitar no Operator AAP

```yaml
# Editar o YAML do custom resource AAP
# OCP Console → Operators → Ansible Automation Platform → <CR> → YAML
spec:
  lightspeed:
    disabled: false
    chatbot_config_secret_name: chatbot-configuration-secret
```

### Verificação

```bash
# Verificar pods do Lightspeed
oc get pods -n aap | grep lightspeed
# myaap-lightspeed-api-*         Running
# myaap-lightspeed-chatbot-api-* Running

# Verificar MCP servers (se configurados)
oc describe pod myaap-lightspeed-chatbot-api-* -n aap | grep "Container"
# Container: ansible-mcp-lightspeed   ← MCP do Gateway ativo
# Container: ansible-mcp-controller   ← MCP do Controller ativo
```

## Variáveis de Inventário (Containerizado — Tech Preview)

Para o deployment containerizado (RHEL/Podman), as variáveis MCP são configuradas no inventory:

```ini
[all:vars]
# Habilitar Lightspeed Intelligent Assistant (Tech Preview)
lightspeed_enabled=true

# Configuração do LLM
lightspeed_chatbot_url=https://api.openai.com/v1
lightspeed_chatbot_model=gpt-4o-mini
lightspeed_chatbot_token=<api_key>
lightspeed_chatbot_llm_provider_type=openai

# MCP server variables (Tech Preview)
lightspeed_aap_gateway_url=https://gateway.empresa.org
lightspeed_aap_controller_url=https://gateway.empresa.org
```

## Trocar o Modelo LLM (sem downtime)

```
Opção 1: Criar novo secret com nome diferente
  → Atualizar spec.lightspeed.chatbot_config_secret_name no Operator
  → Operator redeploy automático

Opção 2: Mesmo nome de secret (requer reinicialização)
  → spec.lightspeed.disabled: true → Save
  → Deletar Ansible Lightspeed operator
  → spec.lightspeed.disabled: false → Save
  → Recriar com novo secret
```

> **Não** atualizar o secret existente diretamente — a lógica de reconciliação do Operator não detecta mudanças no conteúdo do secret.

## Casos de Uso para Consultoria

```
"Quais jobs falharam nas últimas 2 horas?"
"Quantos hosts estão no inventory de produção?"
"Existe algum Job Template para backup de banco de dados?"
"Qual usuário disparou o último job de patching?"
"Mostre os Job Templates da organização Financeiro"
```

O chatbot usa os MCP servers para buscar dados reais em tempo real — não apenas conhecimento estático do treinamento do LLM.
