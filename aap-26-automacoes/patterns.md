# Padrões — AAP 2.6 Automações

## Padrão 1: EDA Webhook com ITSM (GLPI, ServiceNow)

**Cenário:** Sistema ITSM abre chamado → EDA detecta → dispara Job Template automaticamente.

**Implementação:**
1. Configurar webhook no ITSM apontando para URL do EDA (`https://eda.exemplo.org`)
2. Criar Rulebook com `ansible.eda.webhook` como source
3. Usar `action: debug` primeiro para mapear o payload real
4. Definir condição baseada nos campos do payload (`event.payload.category == "provisioning"`)
5. Definir ação `run_job_template` passando dados do payload como `extra_vars`

**Anti-padrão:** Confiar apenas no título do chamado para disparar jobs — usar campos estruturados do payload JSON para condições confiáveis.

---

## Padrão 2: EE por Domínio de Automação

**Cenário:** Diferentes automações têm diferentes dependências Python/collections.

**Recomendação:** Um EE por domínio, não um EE universal:

| EE | Contém | Para automações |
|---|---|---|
| `ee-vmware` | pyvmomi + community.vmware | Criação/gestão de VMs vCenter |
| `ee-satellite` | redhat.satellite | Patch management RHEL |
| `ee-windows` | ansible.windows + pywinrm | Automações Windows |
| `ee-network` | ansible.netcommon + ntc-templates | Backup de rede |
| `ee-nutanix` | SDK Nutanix | Deploy RHEL em Nutanix |

**Anti-padrão:** Um único EE gigante com todas as dependências — build lento, imagem enorme, difícil de manter.

---

## Padrão 3: CaC em Pipeline GitOps

**Fluxo correto:**
```
feature branch → PR com lint automático → review → merge main → apply em staging → aprovação → apply em produção
```

**Estrutura de repositório:**
```
aap-config/
├── environments/
│   ├── staging/vars/
│   └── production/vars/
├── objects/
│   ├── organizations.yml
│   ├── credentials.yml
│   ├── job_templates.yml
│   └── inventories.yml
└── playbooks/
    └── configure_aap.yml
```

**Anti-padrão:** Aplicar CaC diretamente em produção sem staging — mudanças não testadas podem quebrar Job Templates existentes.

---

## Padrão 4: Tratamento de Segredos no CaC

**Regra:** Nunca texto claro em repositório Git.

| Segredo | Solução |
|---|---|
| Senhas do AAP | Ansible Vault no repositório |
| Credenciais de serviço | Ansible Vault + CI/CD secret |
| Tokens de API | Vault (HashiCorp) via ESO ou lookup |
| Certificados | Ansible Vault ou CyberArk |

```yaml
# Correto — vault no YAML
aap_password: !vault |
      $ANSIBLE_VAULT;1.1;AES256...

# Errado — texto claro
aap_password: minhasenha123
```

---

## Padrão 5: Validação Pré/Pós em Automações Críticas

**Para patch management, criação de VMs e certificados:**

```
1. Pre-check  → verificar conectividade, snapshot, espaço em disco
2. Execução   → aplicar mudança
3. Post-check → verificar resultado, serviço running, versão atualizada
4. Rollback   → (em rescue block) reverter se post-check falhar
```

**Anti-padrão:** Pular pré e pós-validações para "ir mais rápido" — uma falha sem rollback automático pode exigir intervenção manual urgente em produção.

---

## Padrão 6: Idempotência em Roles de Infraestrutura

**Regra:** Toda role deve ser safe de re-executar.

```yaml
# Usar módulos declarativos
- ansible.builtin.user: state=present
- ansible.builtin.package: state=present
- ansible.builtin.service: state=started enabled=true
- ansible.builtin.lineinfile: state=present
- ansible.builtin.file: state=directory

# Evitar shell/command sem changed_when
# Se for inevitável:
- shell: meu_comando
  changed_when: "'changed' in result.stdout"
  register: result
```
