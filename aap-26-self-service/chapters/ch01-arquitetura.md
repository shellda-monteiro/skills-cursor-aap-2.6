# Ch01 — Arquitetura e Pré-requisitos

## O que é o Self-Service Automation Portal

Interface web simplificada que torna a automação Ansible acessível a **usuários de negócio** sem conhecimento de playbooks ou do Controller.

- Baseado em **Red Hat Developer Hub (RHDH/Backstage)**
- Incluído na subscription do AAP (sem subscription separada do RHDH)
- Instalado via **Helm chart no OpenShift**
- Conecta ao AAP via **OAuth2**

## Como Funciona

```
1. Administrador configura o portal conectado ao AAP
2. Job Templates do AAP são sincronizados automaticamente para o portal
3. Aparecem como "self-service templates" com formulários step-by-step
4. Usuário de negócio acessa o portal → seleciona template → preenche formulário
5. Portal lança o Job Template correspondente no AAP
6. Usuário acompanha status do job no portal
```

## Restrições

- Apenas **uma instância AAP** por portal
- Apenas **uma organização AAP** por portal
- Requer instalação no **OpenShift** (não disponível em RHEL containerizado)
- Versão AAP suportada: **2.6**

## Arquitetura de Componentes

```
OpenShift Namespace
  ├── self-service-portal (Backstage/RHDH)
  │   ├── Plugin: AAP Sync (sincroniza templates, usuários, times)
  │   ├── Plugin: AAP Jobs (lança jobs via API)
  │   └── Plugin: EE/Collections Manager
  └── PostgreSQL (banco do portal)

Externo:
  ├── AAP Platform Gateway (OAuth2 + API)
  └── Git Repository (custom templates)
```

## Pré-requisitos

| Requisito | Detalhe |
|---|---|
| **OpenShift** | Versão suportada (ver lifecycle page) |
| **Helm** | 3.10 ou superior |
| **oc CLI** | Instalado na máquina de instalação |
| **AAP** | Versão 2.6 com permissão para criar OAuth Application |
| **Permissões OCP** | Criar aplicações no projeto/namespace alvo |
| **Subscription** | AAP (o RHDH está incluído) |
