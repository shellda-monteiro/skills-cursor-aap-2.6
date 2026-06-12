# Glossário — AAP 2.6 Operação e Segurança

**Authenticator**
Método de autenticação configurado no Platform Gateway. Pode ser Local, LDAP, SAML, OIDC, Microsoft Entra ID, Google OAuth2, ou TACACS+. Múltiplos authenticators podem coexistir com prioridade configurável.

**Authenticator Map**
Regra que mapeia atributos do Identity Provider (grupos AD, atributos SAML) para Organizations, Teams e Roles dentro do AAP.

**RBAC (Role-Based Access Control)**
Controle de acesso por funções. No AAP: Organizations → Teams → Users + Roles por tipo de recurso.

**Organization**
Unidade de isolamento no AAP. Todo recurso (inventário, projeto, credencial, job template) pertence a uma organização. Permite segmentar times e projetos.

**Instance Group**
Agrupamento de execution nodes. Permite direcionar Job Templates a conjuntos específicos de nós (ex: prod-only, dmz-network, dev-sandbox).

**System Administrator**
Superusuário do AAP com acesso irrestrito a todos os recursos de todas as organizações.

**System Auditor**
Usuário somente-leitura em toda a plataforma. Útil para auditoria e compliance sem risco de mudanças.

**Activity Stream**
Log imutável de todas as mudanças na plataforma (criação, edição, exclusão de recursos, execuções). Acessível via UI e API.

**LDAP (Lightweight Directory Access Protocol)**
Protocolo para autenticação e busca em diretórios corporativos como Active Directory. Usar LDAPS (porta 636) para comunicação criptografada.

**LDAPS**
LDAP sobre TLS. Porta padrão 636. Obrigatório em produção para evitar transmissão de credenciais em texto claro.

**SAML 2.0 (Security Assertion Markup Language)**
Padrão XML para SSO federado. Usado para integrar AAP com Okta, ADFS, PingFederate, etc.

**Service Provider (SP)**
O AAP no contexto de SAML — é o serviço que consome a autenticação.

**Identity Provider (IdP)**
O sistema que autentica o usuário (Active Directory, Okta, Google, Keycloak).

**OIDC (OpenID Connect)**
Camada de identidade sobre OAuth2. Suportado pelo Platform Gateway para integração com qualquer IdP compatível (Keycloak, Auth0, etc.).

**Microsoft Entra ID**
Novo nome do Azure Active Directory. Suportado nativamente como tipo de authenticator no AAP 2.6.

**OAuth2 Token**
Token de acesso à API do AAP. Pode ser Personal Access Token (PAT) ou token de aplicação OAuth2. Expiração configurável.

**Break-glass Account**
Conta de emergência com acesso de System Administrator para situações onde o LDAP/SAML está inacessível. Deve ser guardada em cofre físico ou cofre de segredos.

**Ansible Vault**
Ferramenta de criptografia para arquivos e strings no Ansible. Obrigatório para senhas no inventory file. Nunca armazenar texto claro em Git.

**CyberArk / HashiCorp Vault / ESO**
Sistemas externos de gestão de segredos integráveis ao AAP para rotação automática de credenciais sem armazenamento no inventory.

**AnsibleAutomationPlatformBackup**
Custom Resource do Operator para criar backups do AAP no OpenShift.

**AnsibleAutomationPlatformRestore**
Custom Resource do Operator para restaurar um backup no OpenShift.
