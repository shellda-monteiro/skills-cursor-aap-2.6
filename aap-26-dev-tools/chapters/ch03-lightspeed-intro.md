# Ch03 — Ansible Lightspeed: Introdução e Requisitos

## O que é o Ansible Lightspeed

**Red Hat Ansible Lightspeed with IBM watsonx Code Assistant** é um serviço de IA generativa integrado ao VS Code que:

- Gera recomendações de **tasks Ansible** a partir de prompts em linguagem natural (inglês)
- Gera **playbooks completos** a partir de intenções descritas em texto
- **Explica** playbooks e tasks existentes
- Utiliza modelos IBM watsonx Granite treinados no ecossistema Ansible (Galaxy, GitHub, coleções certificadas/validadas)

## Benefícios

| Benefício | Descrição |
|---|---|
| **Reduz curva de aprendizado** | Novos usuários Ansible descrevem o objetivo em inglês e recebem código YAML pronto |
| **Aumenta produtividade** | Recomendações aderentes a best practices Ansible + fine-tuning com conteúdo da organização |
| **Conteúdo confiável** | Modelos treinados em conteúdo certificado Red Hat, não em repositórios genéricos |
| **Explica o código** | Gera explicações de playbooks e tasks — útil para revisão e onboarding |

## Cloud Service vs. On-Premise

| | Cloud Service | On-Premise |
|---|---|---|
| **Onde roda** | Serviço Red Hat na nuvem | IBM Cloud Pak for Data (na organização) |
| **Requisitos** | AAP subscription + IBM watsonx Code Assistant subscription (ou trial AAP) | AAP subscription + IBM watsonx Code Assistant for Red Hat Ansible Lightspeed on Cloud Pak for Data |
| **Air-gap** | ✗ Não | ✓ Sim (dados privados) |
| **Versão AAP mínima** | — | 2.4+ |
| **Fine-tuning privado** | ✗ Não | ✓ Sim (modelos treinados no conteúdo da organização) |

## Trial de 90 Dias

Usuários existentes do AAP podem iniciar um trial **gratuito** de 90 dias do Lightspeed cloud service:

- **Não requer** subscription IBM watsonx Code Assistant
- **Requer** subscription (trial ou paga) do AAP
- Permite: recomendações single-task e multi-task, geração de playbooks, explicações

## Recursos Disponíveis

| Recurso | Descrição |
|---|---|
| **Single-task recommendation** | Prompt para uma task → código YAML da task |
| **Multi-task recommendation** | Prompt para múltiplas tasks → bloco de tasks YAML |
| **Playbook generation** | Descrever objetivo → playbook completo gerado |
| **Playbook explanation** | Selecionar playbook existente → explicação em linguagem natural |
| **Task explanation** | Selecionar task → o que ela faz e qual o impacto |
| **Fine-tuning** | Treinar modelo com repositórios Git privados da organização |
| **Ansible Code Bot** | Bot GitHub/GitLab que sugere atualizações automáticas em playbooks (módulos deprecados, etc.) |

## Como o Lightspeed Processa o Contexto

```
1. Desenvolvedor escreve nome da task (comentário YAML) no VS Code
2. Lightspeed lê: nome da task + variáveis do playbook + nome do arquivo
3. Envia para o modelo IBM watsonx Code Assistant
4. Modelo retorna sugestão de código YAML da task
5. Desenvolvedor aceita/rejeita/edita a sugestão
```
