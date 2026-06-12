# Patterns — Mesh, Performance e Upgrade

## Pattern: Segregated Execution (DMZ)

**Problema:** Execution nodes precisam ficar em rede restrita (DMZ) sem conexão de entrada.

```ini
# RPM — execution nodes da DMZ atrás de hop node
[automationcontroller]
ctrl1.exemplo.org
ctrl2.exemplo.org

[automationcontroller:vars]
node_type=control
peers=instance_group_internal  # controller peera só com nodes internos

[instance_group_internal]
exec-internal1.exemplo.org
exec-internal2.exemplo.org

[hop_dmz]
hop-dmz.exemplo.org

[hop_dmz:vars]
node_type=hop
peers=automationcontroller  # hop conecta ao controller

[instance_group_dmz]
exec-dmz1.exemplo.org

[instance_group_dmz:vars]
peers=hop_dmz  # execution DMZ conecta via hop

[execution_nodes]
exec-internal1.exemplo.org
exec-internal2.exemplo.org
hop-dmz.exemplo.org
exec-dmz1.exemplo.org
```

```yaml
# Job Template de rede → restringir ao grupo DMZ
- ansible.platform.job_template:
    name: "Backup Rede"
    instance_groups: ["instance_group_dmz"]
```

---

## Pattern: Outbound-Only Execution Node (Cloud/Firewall restritivo)

**Problema:** Execution node em cloud não pode receber conexões — só pode iniciar.

```ini
[execution_nodes]
exec-cloud.exemplo.org

[execution_nodes:vars]
peers=automationcontroller  # execution inicia a conexão em direção ao controller
```

---

## Pattern: Performance EDA — Alto Volume de Eventos

**Problema:** EDA recebendo milhares de eventos por minuto, banco sobrecarregado.

```yaml
# Rulebook Activation: ativar Skip audit events
# EDA UI → Ativação → Skip audit events: ON

# Para Job Templates disparados pelo EDA:
# - Usar instance_groups dedicado para não interferir em outros jobs
# - Monitorar via: EDA → Analytics
```

---

## Pattern: Upgrade RPM → Container (mesmo hardware)

```bash
# 1. Fazer backup do estado atual
cd /path/to/aap-installer/
./setup.sh --tags backup

# 2. Instalar o Containerized installer no mesmo (ou novo) host
tar xfvz ansible-automation-platform-containerized-setup-2.6.tar.gz
cd ansible-automation-platform-containerized-setup-2.6/

# 3. Configurar inventory com dados do ambiente atual
# (usar senhas, hostnames e configurações do inventário RPM)

# 4. Rodar migração
ansible-playbook -i inventory-migration ansible.containerized_installer.install
```

---

## Pattern: Adicionar Execution Node sem Downtime (OCP)

```
OCP: Automation Execution → Infrastructure → Instances → Add
→ Configurar hostname, porta 27199, tipo=execution
→ Associar ao Instance Group destino
```

Para VM-based/Container — requer re-run do installer:
```bash
# (adicionar novo nó ao grupo [execution_nodes] no inventory antes)
ansible-playbook -i inventory ansible.containerized_installer.install
```
