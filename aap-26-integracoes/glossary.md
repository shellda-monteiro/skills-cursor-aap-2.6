# Glossário — Integrações, Hub e Controller

| Termo | Definição |
|---|---|
| **hashicorp.terraform** | Coleção Red Hat Certified para integrar Ansible com Terraform (gerenciar infraestrutura via Terraform a partir de playbooks). |
| **hashicorp.vault** | Coleção Red Hat Certified para gerenciar e consumir segredos do HashiCorp Vault. Substitui `community.hashi_vault`. |
| **ansible/aap** | Provider Terraform para interagir com a API do Ansible Automation Platform. Permite criar inventories, hosts e disparar jobs. |
| **TF Actions** | Mecanismo do Provider Terraform para disparar eventos no AAP (job launches ou event streams EDA) ao criar/destruir recursos. |
| **External Secret Provider** | Sistema externo de gerenciamento de segredos (Vault, CyberArk, AWS SM, etc.) integrado ao Controller via Credential Plugin. |
| **Credential Plugin** | Interface do Controller que permite buscar secrets em sistemas externos no momento de execução do job. |
| **Secret Lookup Metadata** | Dados adicionais necessários para localizar um segredo no provedor externo (path, região, account name, etc.). |
| **Certified Collection** | Collection com suporte conjunto Red Hat + parceiro. Requer subscrição AAP. Disponível no console.redhat.com. |
| **Validated Collection** | Collection desenvolvida por Red Hat, com testes e validação por especialistas. Complementa as Certified. |
| **Galaxy Collection** | Collection da comunidade Ansible. Sem suporte formal. |
| **Private Automation Hub** | Registry interno de collections e EEs da organização. Substitui acesso direto ao Ansible Galaxy/registry.redhat.io. |
| **Namespace (Hub)** | Espaço de armazenamento único no Hub para publicar e organizar collections internas. |
| **Offline Token** | Token de autenticação de longa duração do console.redhat.com para sync com automation hub. Expira em 30 dias de inatividade. |
| **awx-manage** | Utilitário CLI do Automation Controller. Usado para import de inventory, cleanup, analytics e gerenciamento de cluster. |
| **AnsibleAutomationPlatformBackup** | CRD do Operator OpenShift para disparar backup do AAP. |
| **AnsibleAutomationPlatformRestore** | CRD do Operator OpenShift para restaurar backup do AAP. |
| **must-gather** | Comando `oc adm must-gather` que coleta dados diagnósticos do AAP no OpenShift para suporte. |
| **Signing Service** | Serviço do Hub que assina container images com GPG para garantir integridade e autenticidade dos EEs. |
