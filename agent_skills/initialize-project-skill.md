---
name: initialize-project
description: Initialize and set up a new Solace Agent Mesh project from scratch, including broker configuration, LLM settings, and initial component setup. Use this skill when creating new SAM projects, setting up development environments, or onboarding to the framework.
compatibility: Requires Python 3.10+, pip or uv package manager
metadata:
  author: solace
  version: "1.0"
  category: setup
  complexity: beginner
  estimated_time: 15-30min
---

# Initialize a Solace Agent Mesh Project

This skill guides you through creating a new Solace Agent Mesh (SAM) project from scratch. By the end, you'll have a working multi-agent AI system with a web interface.

## Prerequisites

- Python 3.10 or higher installed
- Access to an LLM API (OpenAI, Azure OpenAI, Google Vertex AI, or compatible)
- One of the following for the message broker:
  - Docker/Podman for local broker (easiest for development)
  - Existing Solace PubSub+ broker credentials
  - Dev mode (in-memory, no broker needed)

## Step 1: Set Up Python Environment

Always use a virtual environment for SAM projects:

```bash
# Create project directory
mkdir my-agent-mesh
cd my-agent-mesh

# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install Solace Agent Mesh
pip install solace-agent-mesh
```

Verify installation:
```bash
sam --version
```

## Step 2: Initialize the Project

SAM provides two initialization methods:

### Option A: GUI Mode (Recommended)

Launch the browser-based configuration wizard:

```bash
sam init --gui
```

This opens a web interface at `http://127.0.0.1:5002` where you can configure:
- LLM provider and API keys
- Broker connection (local container, existing Solace, or dev mode)
- Initial agents and gateways
- Session and artifact storage

### Option B: CLI Mode

For automated setups or when you prefer the terminal:

```bash
sam init
```

Follow the interactive prompts, or use flags for non-interactive setup:

```bash
sam init --skip \
  --llm-service-endpoint "https://api.openai.com/v1" \
  --llm-service-api-key "$OPENAI_API_KEY" \
  --llm-service-planning-model-name "gpt-4o" \
  --llm-service-general-model-name "gpt-4o-mini" \
  --broker-type dev \
  --add-webui-gateway
```

## Step 3: Choose a Broker Configuration

### Development Mode (Simplest)

For quick experimentation without a real broker:

```bash
sam init --dev-mode
```

This uses an in-memory broker - perfect for learning but not for production.

### Local Container Broker (Recommended for Development)

Start a local Solace PubSub+ broker in Docker/Podman:

```bash
sam init --broker-type container --container-engine docker
```

SAM will automatically:
1. Pull the Solace PubSub+ image
2. Start the container
3. Configure connection settings

### Existing Solace Broker

Connect to an existing PubSub+ broker:

```bash
sam init --broker-type solace \
  --broker-url "tcp://your-broker:55555" \
  --broker-vpn "default" \
  --broker-username "admin" \
  --broker-password "your-password"
```

## Step 4: Configure LLM Provider

SAM supports multiple LLM providers. Configure in `.env` or during init:

### OpenAI

```bash
# .env file
LLM_SERVICE_ENDPOINT=https://api.openai.com/v1
LLM_SERVICE_API_KEY=sk-your-key-here
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o
LLM_SERVICE_GENERAL_MODEL_NAME=gpt-4o-mini
```

### Azure OpenAI

```bash
# .env file
LLM_SERVICE_ENDPOINT=https://your-resource.openai.azure.com
LLM_SERVICE_API_KEY=your-azure-key
LLM_SERVICE_PLANNING_MODEL_NAME=your-gpt4-deployment
LLM_SERVICE_GENERAL_MODEL_NAME=your-gpt35-deployment
```

### Google Vertex AI

```bash
# .env file
LLM_SERVICE_ENDPOINT=https://your-region-aiplatform.googleapis.com
LLM_SERVICE_API_KEY=your-service-account-key
LLM_SERVICE_PLANNING_MODEL_NAME=gemini-1.5-pro
LLM_SERVICE_GENERAL_MODEL_NAME=gemini-1.5-flash
```

## Step 5: Verify Project Structure

After initialization, your project should have:

```
my-agent-mesh/
├── .env                      # Environment variables (API keys, broker config)
├── configs/
│   ├── agents/
│   │   └── main_orchestrator.yaml   # Main orchestrator agent
│   ├── gateways/
│   │   └── webui_gateway.yaml       # Web UI gateway (if enabled)
│   └── shared_config.yaml           # Shared configuration
├── logging_config.yaml       # Logging settings
└── custom_components/        # Your custom code (tools, etc.)
```

## Step 6: Run the Application

Start your agent mesh:

```bash
sam run
```

If you enabled the Web UI gateway, open your browser to:
```
http://localhost:8000
```

You should see a chat interface where you can interact with your agent mesh.

## Project Configuration Files

### .env (Environment Variables)

```bash
# Broker Configuration
SOLACE_BROKER_URL=tcp://localhost:55555
SOLACE_BROKER_VPN=default
SOLACE_BROKER_USERNAME=admin
SOLACE_BROKER_PASSWORD=admin

# LLM Configuration
LLM_SERVICE_ENDPOINT=https://api.openai.com/v1
LLM_SERVICE_API_KEY=sk-your-key
LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o
LLM_SERVICE_GENERAL_MODEL_NAME=gpt-4o-mini

# Web UI (if enabled)
FASTAPI_HOST=0.0.0.0
FASTAPI_PORT=8000
```

### configs/shared_config.yaml

```yaml
# Shared settings used by all components
shared_config:
  namespace: ${NAMESPACE:my-app}
  
  # LLM settings
  llm_service:
    endpoint: ${LLM_SERVICE_ENDPOINT}
    api_key: ${LLM_SERVICE_API_KEY}
    planning_model: ${LLM_SERVICE_PLANNING_MODEL_NAME}
    general_model: ${LLM_SERVICE_GENERAL_MODEL_NAME}
  
  # Broker settings  
  broker:
    url: ${SOLACE_BROKER_URL}
    vpn: ${SOLACE_BROKER_VPN}
    username: ${SOLACE_BROKER_USERNAME}
    password: ${SOLACE_BROKER_PASSWORD}
```

## Common Patterns

### Quick Development Setup

```bash
# One-liner for development
python -m venv .venv && source .venv/bin/activate && \
pip install solace-agent-mesh && sam init --gui
```

### CI/CD Non-Interactive Setup

```bash
sam init --skip \
  --llm-service-endpoint "$LLM_ENDPOINT" \
  --llm-service-api-key "$LLM_API_KEY" \
  --llm-service-planning-model-name "gpt-4o" \
  --llm-service-general-model-name "gpt-4o-mini" \
  --broker-type dev \
  --add-webui-gateway \
  --namespace "ci-test"
```

### Multiple Environments

Create environment-specific `.env` files:
```bash
.env.development
.env.staging
.env.production
```

Load the appropriate one:
```bash
cp .env.development .env
sam run
```

## Troubleshooting

### "sam: command not found"

Ensure your virtual environment is activated:
```bash
source .venv/bin/activate
which sam  # Should show path in .venv
```

### Broker Connection Failed

1. Check broker is running: `docker ps` (for container broker)
2. Verify credentials in `.env`
3. Test connection: `telnet localhost 55555`

### LLM API Errors

1. Verify API key is correct in `.env`
2. Check endpoint URL format
3. Ensure model names match your provider's naming

### Port Already in Use

Change the web UI port:
```bash
# In .env
FASTAPI_PORT=8001
```

## Next Steps

After initialization:
1. **Add agents**: `sam add agent my-agent --gui`
2. **Add tools**: Configure built-in or custom tools
3. **Add gateways**: `sam add gateway my-gateway --gui`
4. **Install plugins**: `sam plugin catalog`

## Related Skills

- [configure-shared-settings](configure-shared-settings-skill.md) - Environment and YAML configuration
- [run-and-debug-projects](run-and-debug-projects-skill.md) - Running and monitoring
- [create-and-manage-agents](create-and-manage-agents-skill.md) - Adding custom agents
