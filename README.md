# mcp-server-tableau

> **This repository is a placeholder.** Real implementation will be added in future iterations.

## Overview

This project will contain a **custom MCP (Model Context Protocol) server for Tableau**, designed to be deployed as an agent runtime on **Amazon Bedrock AgentCore**.

The goal is to expose Tableau capabilities (data sources, workbooks, dashboards, and analytics) as tools consumable by AI agents running on the AWS Bedrock platform.

---

## Current State

The code in this repository is based on the [AWS Bedrock AgentCore getting started guide for custom agents](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/getting-started-custom.html). It implements the minimum contract required by AgentCore Runtime:

| Endpoint | Method | Description |
|---|---|---|
| `/invocations` | `POST` | Main agent interaction endpoint |
| `/ping` | `GET` | Health check |

The server is built with **FastAPI** and uses **Strands Agents** as the AI processing engine.

---

## Planned Implementation

Future versions of this repository will include Tableau-specific tooling, such as:

- Querying published data sources
- Listing and filtering workbooks and views
- Extracting insights from dashboards
- Triggering extract refreshes
- Managing Tableau Server / Tableau Cloud resources via the REST API

---

## Architecture

```
AWS Bedrock AgentCore
        │
        ▼
 Agent Runtime (ECR)
        │
        ▼
 FastAPI Server (port 8080)
    ├── POST /invocations  ──► Strands Agent ──► Tableau Tools
    └── GET  /ping
```

**Key components:**

- **FastAPI** — HTTP server adhering to the AgentCore Runtime contract
- **Strands Agents** — AI agent framework handling tool orchestration
- **Docker (ARM64)** — Container image built for `linux/arm64` as required by AgentCore
- **Amazon ECR** — Container registry for the agent image
- **Amazon Bedrock AgentCore** — Managed runtime for hosting the agent

---

## Deployment (current placeholder)

### Prerequisites

- Docker with `buildx` support
- AWS CLI configured
- An ECR repository
- An IAM role with Bedrock AgentCore permissions

### Build and push image

```bash
docker buildx create --use
docker buildx build --platform linux/arm64 \
  -t <account-id>.dkr.ecr.<region>.amazonaws.com/mcp-server-tableau:latest \
  --push .
```

### Deploy to AgentCore

```python
import boto3

client = boto3.client('bedrock-agentcore-control', region_name='<region>')

response = client.create_agent_runtime(
    agentRuntimeName='mcp-server-tableau',
    agentRuntimeArtifact={
        'containerConfiguration': {
            'containerUri': '<account-id>.dkr.ecr.<region>.amazonaws.com/mcp-server-tableau:latest'
        }
    },
    networkConfiguration={"networkMode": "PUBLIC"},
    roleArn='arn:aws:iam::<account-id>:role/AgentRuntimeRole'
)
```

---

## References

- [AWS Bedrock AgentCore – Custom Agent Guide](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/getting-started-custom.html)
- [Strands Agents](https://github.com/strands-agents/sdk-python)
- [Tableau REST API](https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api.htm)
