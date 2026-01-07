# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TAS Workflow Builder** is a microservice for creating and executing multi-step AI workflows within the TAS (Tributary AI System) ecosystem. It provides a visual workflow designer, integration with Argo Workflows for Kubernetes orchestration, and seamless connectivity with TAS services including agents, LLM routing, and document processing.

## Data Models & Schema Reference

### Service-Specific Data Models
This service's data models are comprehensively documented in the centralized data models repository:

**Location**: `../aether-shared/data-models/tas-workflow-builder/`

#### Key Workflow Models:
The data models for TAS Workflow Builder are organized in the following subdirectories:
- **`workflows/`** - Workflow definitions, execution models, and state management
- **`steps/`** - Individual step configurations and execution logic
- **`templates/`** - Pre-built workflow templates for common use cases

These models define the structure for creating, executing, and managing AI-powered workflows.

#### Cross-Service Integration:
- **Platform ERD** (`../aether-shared/data-models/cross-service/diagrams/platform-erd.md`) - Complete entity relationship diagram
- **Architecture Overview** (`../aether-shared/data-models/cross-service/diagrams/architecture-overview.md`) - Workflow Builder in system architecture
- **Document Upload Flow** (`../aether-shared/data-models/cross-service/flows/document-upload.md`) - Document processing workflow example

#### When to Reference Data Models:
1. Before creating new workflow definitions or modifying workflow schemas
2. When implementing new workflow step types or execution logic
3. When debugging workflow execution issues or step failures
4. When onboarding new developers to understand the workflow architecture
5. Before adding new workflow templates or integration patterns

**Main Documentation Hub**: `../aether-shared/data-models/README.md` - Complete navigation for all 38 data model files

## Technology Stack

- **Language**: Go 1.21+ (backend), TypeScript (workflow designer)
- **Database**: PostgreSQL (shared TAS infrastructure)
- **Orchestration**: Argo Workflows (Kubernetes-native workflow engine)
- **Framework**: Gin HTTP framework
- **Message Queue**: Kafka for async workflow execution
- **Monitoring**: Prometheus metrics + Grafana dashboards

## Key Features

### Workflow Designer
- **Visual Builder**: Drag-and-drop interface for workflow creation
- **Step Library**: Pre-built steps for common operations
- **Conditional Logic**: Branching and conditional execution
- **Error Handling**: Automatic retry, fallback, and error recovery

### Multi-Step Execution
- **Sequential Steps**: Execute steps in order with data passing
- **Parallel Execution**: Run multiple steps concurrently
- **Dynamic Steps**: Generate steps based on runtime conditions
- **State Management**: Maintain workflow state across steps

### Argo Workflows Integration
- **Kubernetes Native**: Leverage Argo for scalable workflow orchestration
- **Resource Management**: CPU/memory limits per workflow step
- **Artifact Storage**: MinIO integration for workflow artifacts
- **Workflow Templates**: Reusable Argo workflow templates

### TAS Service Integration
- **Agent Execution**: Invoke TAS Agent Builder agents as workflow steps
- **LLM Routing**: Use TAS-LLM-Router for AI-powered steps
- **Document Processing**: Integrate with AudiModal for document workflows
- **Vector Search**: Query DeepLake for semantic search steps
- **MCP Capabilities**: Leverage TAS MCP servers for extended functionality

## Common Commands

```bash
# Build the application
make build

# Run tests
make test

# Start development server
make dev

# Build Docker image
make docker-build

# Deploy to Kubernetes with Argo
make k8s-deploy

# View workflow logs
make workflow-logs

# List running workflows
make workflow-list
```

## API Endpoints

### Workflow Management
- `POST /workflows` - Create new workflow definition
- `GET /workflows` - List workflows (filtered by space/tenant)
- `GET /workflows/{id}` - Get workflow details
- `PUT /workflows/{id}` - Update workflow definition
- `DELETE /workflows/{id}` - Delete workflow

### Workflow Execution
- `POST /workflows/{id}/execute` - Execute workflow with input
- `GET /executions` - List execution history
- `GET /executions/{id}` - Get execution details and status
- `GET /executions/{id}/logs` - Get execution logs
- `POST /executions/{id}/cancel` - Cancel running execution

### Workflow Templates
- `GET /templates` - List available workflow templates
- `GET /templates/{id}` - Get template details
- `POST /templates/{id}/instantiate` - Create workflow from template

### Argo Integration
- `GET /argo/workflows` - List Argo workflows
- `GET /argo/workflows/{id}` - Get Argo workflow status
- `POST /argo/workflows/{id}/retry` - Retry failed Argo workflow

### Management
- `GET /health` - Health check endpoint
- `GET /metrics` - Prometheus metrics endpoint
- `GET /stats` - Workflow usage statistics

## Workflow Step Types

### AI-Powered Steps
- **Agent Step**: Execute TAS Agent Builder agent
- **LLM Step**: Direct LLM call via TAS-LLM-Router
- **Classification Step**: Document classification with AudiModal
- **Embedding Step**: Generate embeddings with DeepLake

### Data Processing Steps
- **Transform Step**: Data transformation and mapping
- **Filter Step**: Conditional data filtering
- **Aggregate Step**: Data aggregation and summarization
- **Split Step**: Split data for parallel processing

### Integration Steps
- **HTTP Step**: Call external HTTP APIs
- **MCP Step**: Invoke MCP server capabilities
- **Database Step**: Query or update database
- **Kafka Step**: Publish to Kafka topic

### Control Flow Steps
- **Conditional Step**: Branching logic
- **Loop Step**: Iterate over data
- **Parallel Step**: Execute multiple steps concurrently
- **Wait Step**: Delay execution

## Integration Points

- **Argo Workflows**: Kubernetes-native workflow orchestration engine
- **TAS Agent Builder**: Execute agents as workflow steps
- **TAS-LLM-Router**: AI-powered workflow steps with LLM integration
- **AudiModal**: Document processing workflow steps
- **DeepLake API**: Semantic search and embedding generation
- **TAS MCP**: MCP server capabilities as workflow steps
- **Kafka**: Event-driven workflow triggers and notifications
- **PostgreSQL**: Workflow and execution persistence

## Configuration

Configuration is managed via environment variables and `.env` file:

```bash
# Database Configuration
DATABASE_URL=postgresql://tasuser:taspassword@localhost:5432/tas_shared

# Argo Workflows Configuration
ARGO_SERVER_URL=https://argo-workflows.tas-shared.svc.cluster.local:2746
ARGO_NAMESPACE=tas-shared

# Service Integration
AGENT_BUILDER_URL=http://tas-agent-builder:8087
LLM_ROUTER_URL=http://tas-llm-router:8085
AUDIMODAL_URL=http://tas-audimodal:8084
DEEPLAKE_URL=http://tas-deeplake-api:8000

# Server Configuration
SERVER_PORT=8088
SERVER_TIMEOUT=300s
```

## Important Notes

- Workflows can span multiple TAS services for complex AI-powered automation
- Argo Workflows provides Kubernetes-native orchestration with auto-scaling
- Workflow state is persisted in PostgreSQL for recovery and auditing
- Each workflow step can have custom timeout, retry, and error handling policies
- Space-based isolation ensures multi-tenant workflow execution security
- Integration with shared TAS infrastructure via `tas-shared-network` Docker network

## Development Workflow

1. **Start shared infrastructure**: `cd ../aether-shared && ./start-shared-services.sh`
2. **Deploy Argo Workflows**: `kubectl apply -f k8s/argo-workflows.yaml`
3. **Start dependent services**: TAS Agent Builder, LLM Router, etc.
4. **Run database migrations**: `make db-migrate-up`
5. **Start development server**: `make dev`
6. **Access workflow designer**: `http://localhost:8088`

## Example Workflow

```yaml
name: Document Analysis Workflow
description: Analyze uploaded documents with AI
steps:
  - name: upload
    type: audimodal
    action: process_document

  - name: extract
    type: llm
    model: gpt-4
    prompt: Extract key entities from this document

  - name: embed
    type: deeplake
    action: generate_embeddings

  - name: classify
    type: agent
    agent_id: document-classifier

  - name: notify
    type: kafka
    topic: document-processed
```
