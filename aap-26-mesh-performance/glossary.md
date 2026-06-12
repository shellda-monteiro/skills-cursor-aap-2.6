# Glossário — Mesh, Performance e Upgrade

| Termo | Definição |
|---|---|
| **Hybrid node** | Nó que combina control plane + execution. Padrão para `[automationcontroller]` em RPM. |
| **Control node** | Nó somente de controle — project updates, system jobs. Sem execução de playbooks de usuário. |
| **Execution node** | Executa playbooks com ansible-runner em Podman. Parte do execution plane. |
| **Hop node** | Relay de tráfego Receptor. Análogo a jump host. Não executa automação. |
| **Receptor** | Daemon que gerencia a camada de rede do Automation Mesh. Usa TLS e protocolo de overlay. |
| **Peers** | Conexões ponto-a-ponto definidas no inventory (`peers=` / `receptor_peers=`). |
| **Instance Group** | Grupo de execution nodes. Job Templates associados a um grupo rodam apenas naqueles nodes. |
| **Container Group** | Conjunto de pods (OCP) que formam o control plane no deployment Operator. |
| **Mesh Ingress** | Route OCP pela qual VMs externas se conectam ao control plane no OpenShift. |
| **Vertical scaling** | Aumentar CPU/RAM de uma VM ou pod. Requer re-run do installer para re-tuning (VM/Container). |
| **Horizontal scaling** | Adicionar mais nós/replicas. No OCP: sem downtime. Em VM/Container: requer downtime. |
| **Forks** | Número máximo de conexões paralelas de um job. Calculado automaticamente pelo installer. |
| **Job events** | Registros de cada passo de um playbook. Alto verbosity = muito mais eventos = carga maior. |
| **EDA Activation** | Instância de rulebook em execução que monitora fontes de eventos. |
| **Skip audit events** | Opção de ativação EDA que para de gravar eventos em tempo real (reduz carga em alto volume). |
| **Growth topology** | Instalação mínima all-in-one. PoC/pequenos ambientes. Single point of failure. |
| **Enterprise topology** | Instalação com componentes separados. HA, redundância, escalabilidade independente. |
| **In-place upgrade** | Upgrade no mesmo servidor. Suportado entre minor versions. **Não suportado** entre major RHEL versions. |
| **RPM installer** | Método de instalação depreciado desde AAP 2.5. Será removido no AAP 2.7. |
| **Containerized installer** | Método atual para VM-based. Usa Podman + systemd. Suporta RHEL 9 e 10. |
