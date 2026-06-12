# Ch02 — ansible-navigator: Uso Prático e Configuração

## Executar Playbook em um EE Específico

```bash
# Usar EE padrão (auto-detectado)
ansible-navigator run site.yml -i inventory/

# Especificar EE explicitamente
ansible-navigator run site.yml \
  --eei registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest \
  -i inventory/

# Modo stdout (compatível com CI/CD)
ansible-navigator run site.yml -m stdout -i inventory/

# Com extra vars
ansible-navigator run deploy.yml \
  -e "ambiente=producao versao=2.1.0" \
  -m stdout
```

## Explorar EEs Disponíveis

```bash
# Listar EEs (imagens de container Ansible)
ansible-navigator images

# No TUI: navegar para um EE e ver coleções incluídas
# → selecionar EE → Enter → ver lista de coleções
```

## Explorar Coleções de um EE

```bash
# Listar todas as coleções disponíveis no EE ativo
ansible-navigator collections

# Ver documentação de um módulo específico
ansible-navigator doc ansible.builtin.copy
ansible-navigator doc community.vmware.vmware_vm_info
```

## Explorar Inventory

```bash
# Ver estrutura do inventory em modo TUI
ansible-navigator inventory -i inventory/

# No TUI: navegar grupos, hosts, variáveis
# Stdout: exportar inventory como JSON
ansible-navigator inventory -i inventory/ -m stdout --list
```

## Replay de Runs Anteriores

O navigator salva artefatos JSON de cada execução:

```bash
# Replay de run anterior
ansible-navigator replay /tmp/artifacts/minha-run-2025-01-01.json

# No TUI: navegar tasks com falha, ver output host a host
```

## Arquivo de Configuração: ansible-navigator.yml

Colocar na raiz do projeto (`.ansible-navigator.yml`) ou em `~/.ansible-navigator.yml`:

```yaml
---
ansible-navigator:
  ansible:
    inventories:
      - ./inventory
  execution-environment:
    container-engine: podman
    enabled: true
    image: registry.redhat.io/ansible-automation-platform-26/ee-supported-rhel9:latest
    pull:
      policy: missing          # missing | always | never | tag
    volume-mounts:
      - src: "/etc/ssl/certs"
        dest: "/etc/ssl/certs"
        label: "Z"
  logging:
    level: warning
    file: /tmp/navigator.log
  mode: stdout                 # stdout | interactive
  playbook-artifact:
    enable: true
    save-as: /tmp/artifacts/{playbook_name}-{ts_utc}.json
```

### Configurações úteis para times

```yaml
ansible-navigator:
  execution-environment:
    image: registry.empresa.org/aap/ee-corporativo:2.6
    pull:
      policy: missing          # não re-baixar se já existir local
    environment-variables:
      pass:
        - HTTP_PROXY
        - HTTPS_PROXY
        - NO_PROXY
      set:
        ANSIBLE_FORCE_COLOR: "1"
  ansible:
    cmdline: "--forks 15"
  color:
    enable: true
```

### Prioridade de configuração

```
1. Linha de comando (maior prioridade)
2. Variáveis de ambiente (ANSIBLE_NAVIGATOR_*)
3. Arquivo ./ansible-navigator.<yml|yaml|json>   ← projeto
4. ~/.ansible-navigator.<yml|yaml|json>          ← usuário
5. Padrões internos (menor prioridade)
```

## Compartilhar Artefatos de Run

```bash
# Compartilhar resultado de um run com o time (sem reexecutar)
ansible-navigator replay artefato.json

# Útil para: postmortem de falhas, revisão de output, debugging colaborativo
# O artefato inclui: tasks executadas, output por host, facts coletados
```

## Verificar Troubleshooting com Navigator

```bash
# Ver tasks com falha em modo TUI
ansible-navigator run site.yml
# → navegar até task com falha → Enter → ver mensagem de erro completa por host

# Inspecionar configuração do ansible.cfg ativa
ansible-navigator config

# Ver documentação do módulo que falhou
ansible-navigator doc <nome_do_modulo>
```
