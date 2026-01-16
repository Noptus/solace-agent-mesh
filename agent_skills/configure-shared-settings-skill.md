---
name: configure-shared-settings
description: Configure Solace Agent Mesh shared settings including YAML configuration files, environment variables, LLM providers, broker connections, and multi-environment setups. Use this skill when customizing SAM configuration, managing secrets, or setting up different environments.
compatibility: Requires initialized SAM project
metadata:
  author: solace
  version: "1.0"
  category: configuration
  complexity: intermediate
  estimated_time: 20-40min
---

# Configure Shared Settings

This skill covers how to configure Solace Agent Mesh using YAML files, environment variables, and shared configuration patterns. Proper configuration management is essential for development, testing, and production deployments.

## Prerequisites

- Initialized SAM project (`sam init` completed)
- Understanding of YAML syntax
- Access to your LLM provider credentials

## Configuration Architecture

SAM uses a layered configuration approach:

```
┌─────────────────────────────────────┐
│          .env file                  │  ← Secrets & environment-specific
├─────────────────────────────────────┤
│     configs/shared_config.yaml      │  ← Shared across all components
├─────────────────────────────────────┤
│   configs/agents/*.yaml             │  ← Agent-specific settings
│   configs/gateways/*.yaml           │  ← Gateway-specific settings
└─────────────────────────────────────┘
```

## Environment Variables (.env)

The `.env` file stores sensitive values and environment-specific settings:

### Core Variables

```bash
# ===================
# BROKER CONFIGURATION
# ===================
SOLACE_BROKER_URL=tcp://localhost:55555
SOLACE_BROKER_VPN=default
SOLACE_BROKER_USERNAME=admin
SOLACE_BROKER_PASSWORD=admin

# ===================
# LLM CONFIGURATION
# ===================
LLM_SERVICE_ENDPOINT=https://api.openai.com/v1
LLM_SERVICE_API_KEY=sk-your-api-key-here
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o
LLM_SERVICE_GENERAL_MODEL_NAME=gpt-4o-mini

# ===================
# APPLICATION SETTINGS
# ===================
NAMESPACE=my-app
LOG_LEVEL=INFO

# ===================
# WEB UI (if using webui gateway)
# ===================
FASTAPI_HOST=0.0.0.0
FASTAPI_PORT=8000
SESSION_SECRET_KEY=your-secret-key-here
```

### LLM Provider Configurations

#### OpenAI

```bash
LLM_SERVICE_ENDPOINT=https://api.openai.com/v1
LLM_SERVICE_API_KEY=sk-your-openai-key
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o
LLM_SERVICE_GENERAL_MODEL_NAME=gpt-4o-mini
```

#### Azure OpenAI

```bash
LLM_SERVICE_ENDPOINT=https://your-resource.openai.azure.com
LLM_SERVICE_API_KEY=your-azure-api-key
LLM_SERVICE_PLANNING_MODEL_NAME=your-gpt4-deployment-name
LLM_SERVICE_GENERAL_MODEL_NAME=your-gpt35-deployment-name
# Azure-specific
AZURE_API_VERSION=2024-02-15-preview
```

#### Google Vertex AI

```bash
LLM_SERVICE_ENDPOINT=https://us-central1-aiplatform.googleapis.com
LLM_SERVICE_API_KEY=/path/to/service-account.json
LLM_SERVICE_PLANNING_MODEL_NAME=gemini-1.5-pro
LLM_SERVICE_GENERAL_MODEL_NAME=gemini-1.5-flash
# Vertex-specific
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

#### Amazon Bedrock

```bash
LLM_SERVICE_ENDPOINT=https://bedrock-runtime.us-east-1.amazonaws.com
LLM_SERVICE_API_KEY=your-aws-access-key
LLM_SERVICE_PLANNING_MODEL_NAME=anthropic.claude-3-sonnet
LLM_SERVICE_GENERAL_MODEL_NAME=anthropic.claude-3-haiku
# AWS-specific
AWS_SECRET_ACCESS_KEY=your-aws-secret
AWS_REGION=us-east-1
```

## Shared Configuration (shared_config.yaml)

The `configs/shared_config.yaml` file contains settings shared across all components:

### Basic Structure

```yaml
---
# Shared configuration for all SAM components
shared_config:

  # Application namespace
  namespace: ${NAMESPACE:default}
  
  # ========================
  # LLM SERVICE CONFIGURATION
  # ========================
  llm_service:
    endpoint: ${LLM_SERVICE_ENDPOINT}
    api_key: ${LLM_SERVICE_API_KEY}
    
    # Model definitions
    models:
      planning:
        name: ${LLM_SERVICE_PLANNING_MODEL_NAME:gpt-4o}
        temperature: 0.7
        max_tokens: 4096
      general:
        name: ${LLM_SERVICE_GENERAL_MODEL_NAME:gpt-4o-mini}
        temperature: 0.5
        max_tokens: 2048
      
  # ========================
  # BROKER CONFIGURATION
  # ========================
  broker:
    url: ${SOLACE_BROKER_URL:tcp://localhost:55555}
    vpn: ${SOLACE_BROKER_VPN:default}
    username: ${SOLACE_BROKER_USERNAME:admin}
    password: ${SOLACE_BROKER_PASSWORD:admin}
    
  # ========================
  # SESSION MANAGEMENT
  # ========================
  session_service:
    type: memory  # Options: memory, vertex_rag
    behavior: PERSISTENT  # Options: PERSISTENT, RUN_BASED
    
  # ========================
  # ARTIFACT STORAGE
  # ========================
  artifact_service:
    type: filesystem  # Options: memory, filesystem, gcs
    base_path: ./artifacts
    scope: namespace  # Options: namespace, app, custom
```

### Variable Substitution

SAM supports environment variable substitution in YAML:

```yaml
# Basic substitution
api_key: ${LLM_SERVICE_API_KEY}

# With default value
model: ${MODEL_NAME:gpt-4o-mini}

# Nested in strings
endpoint: "https://${HOST}:${PORT}/api"
```

### Advanced LLM Configuration

```yaml
shared_config:
  llm_service:
    endpoint: ${LLM_SERVICE_ENDPOINT}
    api_key: ${LLM_SERVICE_API_KEY}
    
    # Multiple model types for different purposes
    models:
      planning:
        name: ${LLM_SERVICE_PLANNING_MODEL_NAME}
        temperature: 0.7
        max_tokens: 8192
        
      general:
        name: ${LLM_SERVICE_GENERAL_MODEL_NAME}
        temperature: 0.5
        max_tokens: 4096
        
      image_gen:
        name: dall-e-3
        
      multimodal:
        name: gpt-4o
        temperature: 0.3
        
    # Request settings
    request_timeout: 120
    max_retries: 3
    retry_delay: 1
```

### Session and Artifact Configuration

```yaml
shared_config:
  # Session persistence
  session_service:
    type: memory  # In-memory for development
    behavior: PERSISTENT  # Keep sessions across restarts
    # For production, consider:
    # type: vertex_rag
    # corpus_name: your-corpus-name
    
  # File/artifact storage
  artifact_service:
    type: filesystem
    base_path: ${ARTIFACT_PATH:./artifacts}
    scope: namespace
    
    # For cloud storage:
    # type: gcs
    # bucket_name: your-bucket
    # base_path: artifacts/
```

## Agent Configuration

Individual agents are configured in `configs/agents/`:

### Agent YAML Structure

```yaml
---
# configs/agents/my_agent.yaml

# Import shared configuration
shared_config_path: ../shared_config.yaml

# Agent-specific configuration
agent:
  name: my-agent
  
  # Agent instructions (system prompt)
  instruction: |
    You are a specialized agent that helps with data analysis.
    You have access to SQL databases and can run queries.
    Always explain your reasoning before executing queries.
  
  # Which LLM model to use (from shared_config)
  model_type: planning  # Options: planning, general, multimodal
  
  # Enable streaming responses
  supports_streaming: true
  
  # Tool configuration
  tools:
    builtin:
      - artifact_management
      - data_analysis
    custom:
      - path: custom_components/my_tools.py
        class: MyCustomTool
  
  # Agent Card (for discovery)
  agent_card:
    description: "Data analysis agent with SQL capabilities"
    defaultInputModes: ["text"]
    defaultOutputModes: ["text", "file"]
    skills:
      - id: sql_query
        name: SQL Query Execution
        description: "Execute SQL queries on connected databases"
  
  # Agent discovery settings
  discovery:
    enabled: true
    publishing_interval: 30  # seconds
    
  # Inter-agent communication
  inter_agent_communication:
    allow_list: ["*"]  # Allow communication with all agents
    # deny_list: ["restricted-agent"]
    timeout: 60  # seconds
```

### Environment-Specific Agent Settings

Override settings per environment using environment variables:

```yaml
agent:
  name: ${AGENT_NAME:my-agent}
  
  # Production might use different model
  model_type: ${AGENT_MODEL_TYPE:general}
  
  # Different timeouts per environment
  inter_agent_communication:
    timeout: ${IAC_TIMEOUT:60}
```

## Gateway Configuration

Gateways are configured in `configs/gateways/`:

```yaml
---
# configs/gateways/webui_gateway.yaml

shared_config_path: ../shared_config.yaml

gateway:
  name: webui-gateway
  gateway_id: webui-gw-001
  
  # System purpose (context for all requests through this gateway)
  system_purpose: |
    You are a helpful AI assistant accessed through a web interface.
    Be conversational and provide clear, formatted responses.
  
  # How to format responses
  response_format: |
    Format responses in clear markdown with:
    - Headers for sections
    - Code blocks for code
    - Bullet points for lists
  
  # Interface configuration (for HTTP SSE)
  interface:
    type: http_sse
    host: ${FASTAPI_HOST:0.0.0.0}
    port: ${FASTAPI_PORT:8000}
    
  # Artifact handling
  artifact_service:
    type: filesystem
    base_path: ./gateway_artifacts
```

## Multi-Environment Setup

### Directory Structure

```
my-agent-mesh/
├── .env                    # Current environment (gitignored)
├── .env.example            # Template for new developers
├── .env.development        # Development settings
├── .env.staging            # Staging settings
├── .env.production         # Production settings
└── configs/
    ├── shared_config.yaml  # Base configuration
    └── ...
```

### Environment Files

**.env.development**
```bash
SOLACE_BROKER_URL=tcp://localhost:55555
LLM_SERVICE_ENDPOINT=https://api.openai.com/v1
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o-mini  # Cheaper for dev
LOG_LEVEL=DEBUG
NAMESPACE=dev
```

**.env.staging**
```bash
SOLACE_BROKER_URL=tcp://staging-broker.internal:55555
LLM_SERVICE_ENDPOINT=https://api.openai.com/v1
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o
LOG_LEVEL=INFO
NAMESPACE=staging
```

**.env.production**
```bash
SOLACE_BROKER_URL=tcp://prod-broker.internal:55555
LLM_SERVICE_ENDPOINT=https://your-resource.openai.azure.com
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4-turbo
LOG_LEVEL=WARNING
NAMESPACE=production
```

### Switching Environments

```bash
# Development
cp .env.development .env
sam run

# Production
cp .env.production .env
sam run
```

Or use a script:

```bash
#!/bin/bash
# switch-env.sh
ENV=${1:-development}
cp .env.${ENV} .env
echo "Switched to ${ENV} environment"
```

## Logging Configuration

Configure logging in `logging_config.yaml`:

```yaml
---
logging:
  version: 1
  disable_existing_loggers: false
  
  formatters:
    detailed:
      format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    simple:
      format: '%(levelname)s - %(message)s'
    colored:
      class: colorlog.ColoredFormatter
      format: '%(log_color)s%(levelname)s%(reset)s - %(message)s'
  
  handlers:
    console:
      class: logging.StreamHandler
      level: ${LOG_LEVEL:INFO}
      formatter: colored
      stream: ext://sys.stdout
      
    file:
      class: logging.FileHandler
      level: DEBUG
      formatter: detailed
      filename: logs/sam.log
      
  loggers:
    solace_agent_mesh:
      level: ${LOG_LEVEL:INFO}
      handlers: [console, file]
      propagate: false
      
    adk:
      level: WARNING
      handlers: [console]
      propagate: false
  
  root:
    level: ${LOG_LEVEL:INFO}
    handlers: [console]
```

## Common Patterns

### Secrets Management

Never commit secrets to version control:

```bash
# .gitignore
.env
.env.*
!.env.example
```

Use environment variables or secret managers:

```yaml
# Reference secrets from environment
api_key: ${LLM_SERVICE_API_KEY}

# Or use secret manager paths
# api_key: ${vault://secret/llm/api_key}
```

### Configuration Validation

Check configuration before running:

```bash
# Validate YAML syntax
python -c "import yaml; yaml.safe_load(open('configs/shared_config.yaml'))"

# Check environment variables
sam run --dry-run  # (if available)
```

### Feature Flags

Use environment variables for feature toggles:

```yaml
agent:
  tools:
    builtin:
      - artifact_management
      # Conditionally enable based on env var
      # Enable in .env: ENABLE_DATA_TOOLS=true
```

```bash
# .env
ENABLE_EXPERIMENTAL_FEATURES=false
```

## Troubleshooting

### Configuration Not Loading

1. Check YAML syntax:
   ```bash
   python -c "import yaml; yaml.safe_load(open('configs/shared_config.yaml'))"
   ```

2. Verify file paths in agent configs:
   ```yaml
   shared_config_path: ../shared_config.yaml  # Relative to agent file
   ```

### Environment Variables Not Substituted

1. Ensure `.env` file exists and is readable
2. Check variable names match exactly (case-sensitive)
3. Run with verbose logging: `LOG_LEVEL=DEBUG sam run`

### LLM Connection Issues

1. Test API connectivity:
   ```bash
   curl -H "Authorization: Bearer $LLM_SERVICE_API_KEY" \
        "$LLM_SERVICE_ENDPOINT/models"
   ```

2. Verify model names match provider's naming convention

### Broker Connection Issues

1. Test broker connectivity:
   ```bash
   telnet localhost 55555
   ```

2. Check credentials in `.env`

## Related Skills

- [initialize-project](initialize-project-skill.md) - Project setup
- [run-and-debug-projects](run-and-debug-projects-skill.md) - Running with different configs
- [create-and-manage-agents](create-and-manage-agents-skill.md) - Agent configuration details
