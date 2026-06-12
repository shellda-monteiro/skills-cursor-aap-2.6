# Ch03 — Uso: Criar Projetos e Operar no Controller

## Dashboard dos Plug-ins Ansible

Após instalação e login no RHDH:

```
RHDH → ícone Ansible (A) na navegação lateral
  → Overview: painel principal com os 5 passos (Learn → Operate)
```

## 1. Learn — Trilhas de Aprendizado

```
Overview → Learn
  → Learning Paths: links para trilhas em developers.redhat.com
    - Foundations of Ansible
    - Ansible VS Code extension
    - Working with YAML
  → Labs: laboratórios práticos hands-on
```

> Requer conta Red Hat e acesso a developers.redhat.com.

## 2. Discover — Explorar Coleções

```
Overview → Discover Existing Collections
  → Se Private Hub configurado: link direto ao hub interno
  → Se não configurado: link para console.redhat.com/ansible/automation-hub
```

## 3. Create — Criar Projeto Ansible no Git

```
Overview → Create → Create Ansible Git Project → Create Ansible Playbook project

Formulário:
  - Source code repository org/username: minha-org
  - Playbook repository name: patching-rhel
  - Description: Patching RHEL via Satellite
  - Collection namespace: empresa
  - Collection name: infra
  - Catalog Owner Name: time-infra
→ Next → Review → Create Repository
```

**Resultado:** repositório Git criado com:
```
patching-rhel/
├── playbooks/
│   └── site.yml          # playbook inicial com boas práticas
├── collections/
│   └── requirements.yml
├── inventory/
├── ansible-navigator.yml  # pré-configurado com EE corporativo
├── .devfile.yaml          # para Dev Spaces
└── README.md
```

## 4. Develop — IDE Web com Dev Spaces

```
Overview → Develop → Open in Dev Spaces
  → Abre o repositório no OpenShift Dev Spaces (VS Code no browser)
  → Pré-configurado com:
     - ansible-navigator (apontando para EE corporativo)
     - Ansible VS Code extension
     - Lightspeed (se configurado nos plug-ins)
     - Acesso ao repositório Git
```

**Não requer instalação local** — desenvolvedor usa apenas o browser.

## 5. Operate — Configurar e Executar no Controller

```
Overview → Operate → Go to Ansible Automation Platform
  → Abre o Controller AAP
  
No Controller:
  1. Projects → Add → Source Control URL: <URL do repositório Git criado>
  2. Job Templates → Add → associar ao projeto
  3. Launch → executar o playbook desenvolvido
```

## Exemplo Completo: Automatizar Firewall RHEL

```
Passo 1 — Learn:
  RHDH → Ansible → Learn
  → Completar lab "Ansible network automation"

Passo 2 — Create:
  RHDH → Ansible → Create → "Create Ansible Playbook project"
  → Nome: rhel-firewall
  → Repositório criado com estrutura padrão

Passo 3 — Develop:
  RHDH → Ansible → Develop → Open in Dev Spaces
  → Editar playbooks/configure_firewall.yml:
    - name: Configure firewall rules on RHEL
      ansible.posix.firewalld:
        service: "{{ item }}"
        permanent: true
        state: enabled
      loop: "{{ allowed_services }}"

Passo 4 — Operate:
  AAP → Projects → Add → rhel-firewall (repositório Git)
  → Job Template "Configure RHEL Firewall"
  → Launch → selecionar inventory → executar
```
