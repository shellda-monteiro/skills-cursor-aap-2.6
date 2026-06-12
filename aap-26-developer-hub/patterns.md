# Patterns — Ansible Plug-ins para Developer Hub

## Pattern 1: Portal para Time de Automação (Completo)

**Cenário:** organização quer padronizar como desenvolvedores criam projetos Ansible.

```
Arquitetura:
  RHDH (existente ou novo)
    + Ansible Plug-ins
    + Integração: Private Hub, Dev Spaces, Controller

Fluxo:
  Novo desenvolvedor:
    1. RHDH → Ansible → Learn → completa trilha de onboarding
    2. RHDH → Ansible → Create → cria repositório Git com template corporativo
    3. RHDH → Ansible → Develop → abre Dev Spaces → EE corporativo pré-carregado
    4. Desenvolve e testa localmente no Dev Spaces
    5. Git push → CI/CD valida (ansible-lint, molecule)
    6. RHDH → Ansible → Operate → Controller sincroniza o projeto
    7. Cria Job Template → executa em produção
```

---

## Pattern 2: RHDH + Self-Service Portal (Coexistência)

```
Organização com dois portais para públicos diferentes:

RHDH + Ansible Plug-ins (portal.desenvolvimento.empresa.org)
  → Público: time de automação, desenvolvedores
  → Função: criar, aprender, desenvolver

Self-Service Portal (portal.autoatendimento.empresa.org)
  → Público: analistas, gerentes, usuários de negócio
  → Função: executar automações prontas

Infraestrutura compartilhada:
  → Mesmo AAP 2.6 (mesmo Gateway, mesmo Controller)
  → Mesmo Private Automation Hub
  → Git repositories separados por projeto
```

---

## Pattern 3: Onboarding Padronizado de Projetos

```bash
# Software Template criado pelo time de plataforma:
# "Novo Projeto de Automação Corporativo"
# → cria repositório com:
#   - ansible-navigator.yml apontando para EE corporativo
#   - ansible-lint.yml com regras da organização
#   - .gitlab-ci.yml com pipeline de lint + teste
#   - README.md com guia de contribuição
#   - collections/requirements.yml com coleções padrão

# Resultado: qualquer desenvolvedor cria projeto em 2 minutos
# já seguindo todos os padrões da organização
```

---

## Pattern 4: Dev Spaces como Ambiente Padrão (Sem Config Local)

```
Benefício: o desenvolvedor não precisa configurar máquina local

Dev Spaces workspace (configurado pelo .devfile.yaml):
  image: registry.empresa.org/aap/ee-corporativo:2.6
  extensions:
    - redhat.ansible
    - redhat.vscode-commons
  env:
    ANSIBLE_NAVIGATOR_IMAGE: registry.empresa.org/aap/ee-corporativo:2.6
    ANSIBLE_COLLECTIONS_PATH: /home/user/.ansible/collections

→ Desenvolvedor abre RHDH no browser
→ Clica "Open in Dev Spaces"
→ VS Code abre no browser com tudo pré-configurado
→ ansible-navigator run → playbook executa no EE correto
→ Testa antes de fazer push
```
