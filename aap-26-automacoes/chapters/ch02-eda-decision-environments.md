# Ch02 — Decision Environments e Credenciais EDA

## Decision Environments

Decision Environments são imagens de contêiner que contêm o motor de execução dos Rulebooks do EDA. São análogos aos Execution Environments do Automation Controller.

### DE padrão Red Hat (AAP 2.6 — usar rhel9)
```
registry.redhat.io/ansible-automation-platform-26/de-minimal-rhel9:latest
registry.redhat.io/ansible-automation-platform-26/de-supported-rhel9:latest
```

> **IMPORTANTE:** Usar a imagem correspondente à versão do AAP conectado:
> - AAP 2.4 → `de-minimal-rhel9:latest` (platform-24) ou rhel8
> - AAP 2.5 → `de-minimal-rhel9:latest` (platform-25) ou rhel8
> - AAP 2.6 → `de-minimal-rhel9:latest` (platform-26) **apenas rhel9**

### Construir DE customizado (com Ansible Builder ≥ 3.0)

```yaml
# decision-environment.yml
version: 3

images:
  base_image:
    name: 'registry.redhat.io/ansible-automation-platform-26/de-minimal-rhel9:latest'

dependencies:
  galaxy:
    collections:
      - name: servicenow.itsm
  python_interpreter:
    package_system: "python3.12"
  exclude:
    all_from_collections:
      # ansible.eda já está instalado no de-minimal — não reinstalar
      - ansible.eda
  options:
    package_manager_path: /usr/bin/microdnf
```

Para incluir pacotes Python e RPM adicionais:

```yaml
version: 3
images:
  base_image:
    name: 'registry.redhat.io/ansible-automation-platform-26/de-minimal-rhel9:latest'
dependencies:
  galaxy:
    collections:
      - name: servicenow.itsm
  python:
    - six
    - psutil
  python_interpreter:
    package_system: "python3.12"
  exclude:
    all_from_collections:
      - ansible.eda
  options:
    package_manager_path: /usr/bin/microdnf
```

```bash
# Pré-requisito
pip3 install --user 'ansible-builder>=3.0'

# Construir a imagem
ansible-builder build \
  --file decision-environment.yml \
  --tag meu-de:1.0 \
  --container-runtime podman

# Push para Private Hub
podman tag meu-de:1.0 <hub-fqdn>/organizacao/meu-de:1.0
podman push <hub-fqdn>/organizacao/meu-de:1.0
```

## Credenciais no EDA Controller

O EDA usa credenciais para:
1. Sincronizar Projetos Git (Rulebooks)
2. Autenticar com o Automation Controller (para `run_job_template`)
3. Autenticar para baixar Decision Environments do Hub

### Tipos de credenciais no EDA

| Tipo | Uso |
|---|---|
| `Source Control` | Acesso a repositório Git de Rulebooks |
| `Container Registry` | Pull de Decision Environments de registry privado |
| `Red Hat Ansible Automation Platform` | Conexão EDA → Controller para disparar jobs |
| `Vault` | Acesso a HashiCorp Vault para segredos |

### Configurar conexão EDA → Controller

```yaml
# Credential Type: Red Hat Ansible Automation Platform
# Campos obrigatórios:
URL: https://controller.exemplo.org
Username: admin
Password: <senha>
# OU usar OAuth Token em vez de username/password
```

## Variáveis de Ambiente nos Rulebooks

```yaml
- name: Rulebook com segredos injetados
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: processar evento
      condition: event.payload is defined
      action:
        run_job_template:
          name: "Meu Template"
          organization: "Ops"
          job_args:
            extra_vars:
              # Variáveis de ambiente são injetadas automaticamente
              # via Rulebook Activation → Extra Vars
              api_token: "{{ lookup('env', 'API_TOKEN') }}"
```

## Troubleshooting de Ativações EDA

### Diagnóstico sistemático (4 passos)

1. **Checar logs de ativação via UI:**
   - EDA Controller → Automation Decisions → Rulebook Activations → `[nome]` → History
   - Selecionar execução → aba Output

2. **Nível de log Debug:**
   - Se o log padrão (`Error`) não mostrar o problema, recriar a ativação com `Log level: Debug`

3. **Logs por container (acesso ao host):**
   ```bash
   # Ver logs do serviço EDA
   journalctl -u automation-eda-activations -f

   # Arquivos de log no host
   ls /var/log/ansible-automation-platform/eda/

   # Coletar todos os logs para suporte Red Hat
   sosreport  # ou mustgather
   ```

4. **Usar `tid` (transaction ID)** para rastrear uma ativação em múltiplos serviços

### Problema: Ativação presa em estado Pending

**Causas prováveis:**
- Limite de memória/CPU atingido → verificar se há outras ativações Running
- Worker, Redis ou activation-worker não estão rodando
- Erro interno nos containers: `eda-server worker`, `scheduler`, `API` ou `nginx`

**Ação:** terminar ativações desnecessárias e checar containers com `podman ps`

### Problema: Ativação reinicia repetidamente

**Restart Policy disponíveis:**
- `On failure`: reinicia só quando o container falha
- `Always`: sempre reinicia (máx 5 vezes)
- `Never`: nunca reinicia

**Diagnóstico:**
1. Confirmar que Restart Policy = `On failure`
2. Checar o código YAML do rulebook e os logs de instância
3. Mudar Log level para `Debug` e re-executar → analisar Output na aba History

### Problema: Eventos recebidos mas ações não disparam

1. Revisar condições (`when`) no rulebook — devem corresponder exatamente ao payload real
2. Verificar indentação e sintaxe YAML
3. Confirmar que a `action` está corretamente configurada (ex: `run_job_template` com argumentos válidos)

### Problema: Event Streams não enviam eventos para a ativação

- Verificar que Event Stream não está em **Test mode**
- Confirmar que o serviço de origem envia a requisição corretamente
- Checar conectividade com a instância do Platform Gateway
- Verificar que o `event stream worker` está ativo
- Confirmar que a credencial do Event Stream está correta e atualizada no serviço de origem
- Se usando certificados auto-assinados: desabilitar validação de cert no serviço de origem (para testes)

### Problema: Webhook não recebido

```
Origem (GLPI/ITSM) → EDA Controller : TCP 443 (ou porta do source plugin)
```

Verificar:
- Firewall entre origem e EDA
- Porta do source plugin aberta no host EDA
- Credencial de autenticação do webhook configurada na ativação

## Payload de Webhook — Inspecionar Estrutura

Ao integrar com novos sistemas (como GLPI), use `debug` primeiro para ver o payload:

```yaml
- name: Debug GLPI webhook
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: Logar todos os eventos
      condition: event is defined
      action:
        debug:
          msg: |
            Headers: {{ event.meta.headers }}
            Payload: {{ event.payload | to_nice_json }}
```

Após identificar a estrutura real do payload, criar as condições e ações corretas.
