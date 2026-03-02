# mcp-server-tableau

> **Este repositório é um marcador (placeholder).** A implementação real será adicionada em iterações futuras.

## Visão Geral

Este projeto conterá um **servidor MCP (Model Context Protocol) personalizado para o Tableau**, projetado para ser implantado como um runtime de agente no **Amazon Bedrock AgentCore**.

O objetivo é expor as capacidades do Tableau (fontes de dados, pastas de trabalho, painéis e análises) como ferramentas consumíveis por agentes de IA executados na plataforma AWS Bedrock.

---

## Estado Atual

O código neste repositório é baseado no [guia de introdução do AWS Bedrock AgentCore para agentes personalizados](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/getting-started-custom.html). Ele implementa o contrato mínimo exigido pelo AgentCore Runtime:

| Endpoint | Método | Descrição |
|---|---|---|
| `/invocations` | `POST` | Endpoint principal de interação com o agente |
| `/ping` | `GET` | Verificação de integridade (health check) |

O servidor é construído com **FastAPI** e utiliza **Strands Agents** como mecanismo de processamento de IA.

---

## Implementação Planejada

Versões futuras deste repositório incluirão ferramentas específicas do Tableau, tais como:

- Consultar fontes de dados publicadas
- Listar e filtrar pastas de trabalho e exibições (views)
- Extrair insights de painéis (dashboards)
- Acionar atualizações de extratos
- Gerenciar recursos do Tableau Server / Tableau Cloud via API REST

---

## Arquitetura

```
AWS Bedrock AgentCore
        │
        ▼
 Agent Runtime (ECR)
        │
        ▼
 FastAPI Server (port 8080)
    ├── POST /invocations  ──► Strands Agent ──► Ferramentas do Tableau
    └── GET  /ping
```

**Componentes principais:**

- **FastAPI** — Servidor HTTP aderente ao contrato do AgentCore Runtime
- **Strands Agents** — Framework de agente de IA que lida com a orquestração de ferramentas
- **Docker (ARM64)** — Imagem de contêiner construída para `linux/arm64` conforme exigido pelo AgentCore
- **Amazon ECR** — Registro de contêiner para a imagem do agente
- **Amazon Bedrock AgentCore** — Runtime gerenciado para hospedar o agente

---

## Implantação (marcador atual)

### Pré-requisitos

- Docker com suporte a `buildx`
- AWS CLI configurada
- Um repositório ECR
- Uma função (role) IAM com permissões para o Bedrock AgentCore

### Construir e enviar a imagem (push)

```bash
docker buildx create --use
docker buildx build --platform linux/arm64 \
  -t <id-da-conta>.dkr.ecr.<regiao>.amazonaws.com/mcp-server-tableau:latest \
  --push .
```

### Implantar no AgentCore

```python
import boto3

client = boto3.client('bedrock-agentcore-control', region_name='<regiao>')

response = client.create_agent_runtime(
    agentRuntimeName='mcp-server-tableau',
    agentRuntimeArtifact={
        'containerConfiguration': {
            'containerUri': '<id-da-conta>.dkr.ecr.<regiao>.amazonaws.com/mcp-server-tableau:latest'
        }
    },
    networkConfiguration={"networkMode": "PUBLIC"},
    roleArn='arn:aws:iam::<id-da-conta>:role/AgentRuntimeRole'
)
```

---

## Referências

- [AWS Bedrock AgentCore – Guia de Agente Personalizado](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/getting-started-custom.html)
- [Strands Agents](https://github.com/strands-agents/sdk-python)
- [API REST do Tableau](https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api.htm)

---

*Nota: Este repositório está em desenvolvimento ativo.*
