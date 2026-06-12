# Cheatsheet — Ansible Plug-ins para Developer Hub

## Pré-requisitos

```
✓ RHDH instalado no OCP (Helm ou Operator)
✓ Subscription AAP
✓ Acesso ao registry.redhat.io (ou mirror interno)
✓ AAP API Token (read) para integração
```

## Métodos de entrega

| Método | Quando usar |
|---|---|
| **OCI (recomendado)** | RHDH com acesso ao registry.redhat.io |
| **HTTP plug-in registry** | Ambiente air-gapped / sem acesso externo |

## ConfigMap mínimo

```yaml
ansible:
  rhaap:
    baseUrl: 'https://gateway.empresa.org'
    token: '<AAP_API_TOKEN>'
    checkSSL: true
  automationHub:
    baseUrl: 'https://hub.empresa.org'      # opcional
  devSpaces:
    baseUrl: 'https://devspaces.empresa.org' # opcional
```

## Workflow do desenvolvedor

```
Learn (trilhas) → Discover (coleções) → Create (projeto Git)
→ Develop (Dev Spaces) → Operate (Controller)
```

## Estrutura de projeto criada pelo template

```
projeto/
├── playbooks/          # playbooks
├── collections/        # requirements.yml
├── inventory/          # inventories
├── ansible-navigator.yml  # pré-configurado
├── .devfile.yaml       # Dev Spaces config
└── README.md
```

## Diferença dos produtos

| | Ansible Plug-ins | Self-Service Portal |
|---|---|---|
| Público | Desenvolvedores | Usuários de negócio |
| Ação | Criar conteúdo novo | Executar automações |
| RHDH | Plug-in no existente | RHDH dedicado |
