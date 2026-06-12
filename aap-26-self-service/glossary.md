# Glossário — Self-Service Automation Portal

**Self-Service Automation Portal** — Interface web simplificada baseada em Red Hat Developer Hub (RHDH/Backstage) que expõe Job Templates do AAP como formulários ponto-e-clique para usuários de negócio.

**Red Hat Developer Hub (RHDH)** — Plataforma IDP (Internal Developer Platform) baseada em Backstage que serve de base técnica para o Self-Service Portal. Incluída na subscription do AAP.

**Backstage** — Framework open-source da Spotify que o RHDH implementa. O portal usa plugins Backstage para integração com o AAP.

**Custom Self-Service Template** — Template personalizado criado em YAML (catalog-info.yaml) em um repositório Git que define formulário customizado e associa a um Job Template do AAP.

**AAP Sync / Synchronization** — Processo que importa organizações, usuários, times e Job Templates do AAP para o portal. Roda em schedule configurável.

**surveyEnabled filter** — Filtro de sincronização que inclui apenas Job Templates com survey habilitado (`true`), sem survey (`false`), ou todos (omitido).

**RBAC de Template** — Controle de acesso configurado no portal que restringe quais times/grupos podem ver e executar cada template (custom ou auto-gerado).

**Helm chart** — Pacote de configuração que define como o portal é implantado no OpenShift. Configurado via `values.yaml`.

**ImageContentSourcePolicy** — Recurso do OpenShift que redireciona pulls de imagens de um registry público para um registry interno (usado em ambientes air-gapped).

**Redirect URI** — URL configurada na OAuth Application do AAP para onde o usuário é redirecionado após autenticação. Deve ser `https://<URL_PORTAL>/api/auth/aap/handler/frame`.
