# Padrões — AAP 2.6 Operação e Segurança

## Padrão 1: LDAP com Authenticator Maps

**Cenário:** Mapear grupos do Active Directory para Organizations e Teams do AAP.

**Fluxo:**
```
AD Group "CN=Ansible-Admins" → Organization "TI-Infra" → Role: Admin
AD Group "CN=Ansible-Devs"   → Organization "TI-Infra" → Role: Member → Team "Dev"
AD Group "CN=Ansible-Ops"    → Organization "TI-Infra" → Role: Member → Team "Ops"
```

**Anti-padrão:** Dar `System Administrator` a todos os usuários LDAP — perda completa de granularidade de acesso.

---

## Padrão 2: Break-glass Account

**Regra:** Sempre manter um usuário local habilitado como contingência para falha do LDAP/SAML.

```
break-glass-admin → Local Authenticator → System Administrator
Senha: guardada em cofre físico ou cofre de segredos (CyberArk, 1Password)
Uso: SOMENTE quando LDAP/SAML está inacessível
```

**Anti-padrão:** Desabilitar o Local Authenticator completamente — se o LDAP cair, ninguém consegue logar.

---

## Padrão 3: RBAC por Projeto de Automação

**Segmentar por equipe e criticidade:**

```
Organization: "TI-Infraestrutura"
├── Team "Linux-Ops"
│   ├── Execute: Job Template "Patch RHEL"
│   └── Use: Inventory "Servidores-Linux"
├── Team "VMware-Ops"  
│   ├── Execute: Job Template "Criar VM"
│   └── Admin: Inventory "vCenter"
└── Team "Network-Ops"
    ├── Execute: Job Template "Backup Rede"
    └── Use: Inventory "Ativos-Rede"
```

---

## Padrão 4: Credenciais em Vault Externo

**Em vez de armazenar senhas no AAP:**
```
Job Template → Vault Credential → HashiCorp Vault → senha atual
                                                    (rotação automática)
```

**Anti-padrão:** Credenciais com senhas estáticas armazenadas no Controller sem rotação.

---

## Padrão 5: Backup Antes de Qualquer Operação Disruptiva

```
Antes de upgrade    → backup
Antes de restore    → backup do estado atual
Antes de mudança CaC significativa → backup
Antes de upgrade de OS do host     → backup
```

```bash
ansible-playbook -i inventory ansible.containerized_installer.backup \
  -e "backup_dir=/mnt/nfs/aap-backups/"
```

---

## Padrão 6: Ciclo de Upgrade Seguro

```
1. Testar upgrade em staging
2. Verificar changelog de breaking changes
3. Backup de produção
4. Janela de manutenção comunicada
5. Upgrade em produção
6. Smoke test pós-upgrade (jobs críticos)
7. Rollback se smoke test falhar
```

---

## Padrão 7: Instance Groups por Zona de Segurança

```
Instance Group "prod"         → Execution nodes RHEL prod (sem internet)
Instance Group "dmz"          → Execution nodes DMZ (acesso a ativos de rede)
Instance Group "cloud"        → Execution nodes com acesso a APIs cloud
Instance Group "windows"      → Execution nodes com WinRM configurado
```

Cada Job Template usa apenas o Instance Group necessário — mínimo privilégio para execução.
