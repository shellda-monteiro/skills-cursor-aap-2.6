# Patterns — Dev Tools

## Pattern 1: Workflow de Desenvolvimento com EE Corporativo

```bash
# 1. Configurar ansible-navigator.yml no repositório
cat > ansible-navigator.yml << 'EOF'
---
ansible-navigator:
  execution-environment:
    image: registry.empresa.org/aap/ee-corporativo:2.6
    pull:
      policy: missing
    environment-variables:
      pass:
        - HTTP_PROXY
        - HTTPS_PROXY
  mode: stdout
  playbook-artifact:
    enable: true
    save-as: ./artifacts/{playbook_name}-{ts_utc}.json
  ansible:
    inventories:
      - ./inventory
EOF

# 2. Desenvolver e testar localmente com o mesmo EE do Controller
ansible-navigator run site.yml --check -m stdout

# 3. Executar em produção (mesmo comando, mesmo resultado)
ansible-navigator run site.yml -m stdout

# 4. Se falhar: replay para debug
ansible-navigator replay ./artifacts/site-*.json
```

---

## Pattern 2: Lightspeed para Onboarding de Novos Desenvolvedores

**Cenário:** novo membro do time precisa criar um playbook de provisionamento de VM.

```yaml
# Desenvolvedor inicia um arquivo vm-provision.yml
# Digita os nomes das tasks em inglês:

---
- name: Provision VM in VMware vCenter
  hosts: localhost
  tasks:
    - name: Create VM with 4 vCPU 8GB RAM in cluster Prod  # → Lightspeed sugere vmware_guest
    - name: Wait for VM to be powered on and get IP         # → Lightspeed sugere wait_for
    - name: Add VM to inventory group webservers            # → Lightspeed sugere add_host
    - name: Register VM with Red Hat Satellite              # → Lightspeed sugere redhat_subscription
```

**Resultado:** novo desenvolvedor gera 80% do código via sugestões IA, foca em revisar e ajustar.

---

## Pattern 3: Inspecionar EE antes de Deploy

```bash
# Verificar se a coleção necessária está no EE
ansible-navigator collections \
  --eei registry.empresa.org/aap/ee-corporativo:2.6

# No TUI: localizar community.vmware → verificar versão
# Se não estiver: reconstruir o EE com a coleção ou usar outro EE

# Ver documentação do módulo dentro do EE
ansible-navigator doc community.vmware.vmware_guest \
  --eei registry.empresa.org/aap/ee-corporativo:2.6
```

---

## Pattern 4: Debug Colaborativo via Artefato

```bash
# Desenvolvedor A executa o playbook e o run falha
ansible-navigator run deploy.yml -m stdout
# Artefato salvo em: ./artifacts/deploy-2025-01-15T10:30:00.json

# Desenvolvedor A envia o artefato para o time
git add artifacts/deploy-*.json
git commit -m "debug: artefato do run com falha no task X"
git push

# Desenvolvedor B reproduz e inspeciona o run sem reexecutar
git pull
ansible-navigator replay ./artifacts/deploy-2025-01-15T10:30:00.json
# → TUI mostra tasks, navegar até a falha, ver output completo por host
```
