---
name: manage-plugins
description: Discover, install, create, and publish plugins for Solace Agent Mesh. Use this skill when extending SAM with community plugins, creating reusable components, or packaging agents and gateways for distribution.
compatibility: Requires initialized SAM project
metadata:
  author: solace
  version: "1.0"
  category: development
  complexity: intermediate
  estimated_time: 30-45min
---

# Manage Plugins

This skill covers how to work with plugins in Solace Agent Mesh. Plugins are the primary mechanism for extending SAM with pre-built agents, gateways, and tools that can be shared and reused across projects.

## Prerequisites

- Initialized SAM project (`sam init` completed)
- For creating plugins: Python packaging knowledge
- For publishing: PyPI account or internal package repository

## Understanding Plugins

### What is a Plugin?

A plugin in SAM is:
- A packaged, reusable component (agent, gateway, or custom functionality)
- Distributed as a Python wheel package
- Installable via pip or the SAM CLI
- Configurable through YAML without modifying code

### Plugin Types

| Type | Description | Use Case |
|------|-------------|----------|
| Agent | Pre-built AI agent with specific capabilities | Add specialized agents (SQL, RAG, etc.) |
| Gateway | Interface for external systems | Add Slack, Teams, Event Mesh connectivity |
| Custom | Tools, services, or other functionality | Add shared utilities or integrations |

### Plugin Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Plugin Package                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────────┐  ┌──────────────────────────┐ │
│  │   Component      │  │     Configuration        │ │
│  │   (Python)       │  │     Template (YAML)      │ │
│  └──────────────────┘  └──────────────────────────┘ │
│                                                      │
│  ┌──────────────────┐  ┌──────────────────────────┐ │
│  │   Dependencies   │  │     Documentation        │ │
│  │   (pyproject)    │  │     (README)             │ │
│  └──────────────────┘  └──────────────────────────┘ │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## Discovering Plugins

### Plugin Catalog

Browse available plugins with the visual catalog:

```bash
sam plugin catalog
```

This launches a web interface at `http://localhost:5003` showing:
- Official Solace plugins
- Community plugins
- Plugin descriptions and documentation
- One-click installation

### Official Core Plugins

Available at [github.com/SolaceLabs/solace-agent-mesh-core-plugins](https://github.com/SolaceLabs/solace-agent-mesh-core-plugins):

| Plugin | Type | Description |
|--------|------|-------------|
| `sam-slack-gateway` | Gateway | Slack bot integration |
| `sam-teams-gateway` | Gateway | Microsoft Teams integration |
| `sam-event-mesh-gateway` | Gateway | External broker connectivity |
| `sam-sql-agent` | Agent | SQL database querying |
| `sam-rag-agent` | Agent | Retrieval-augmented generation |
| `sam-web-agent` | Agent | Web browsing and scraping |

## Installing Plugins

### From Plugin Catalog

```bash
# Using plugin name from catalog
sam plugin install sam-slack-gateway
```

### From Git Repository

```bash
# From GitHub
sam plugin install https://github.com/SolaceLabs/solace-agent-mesh-core-plugins.git

# From specific branch/tag
sam plugin install https://github.com/org/repo.git@v1.0.0
```

### From Local Path

```bash
# From local directory
sam plugin install /path/to/my-plugin

# From wheel file
sam plugin install /path/to/my_plugin-1.0.0-py3-none-any.whl
```

### Custom Install Command

For environments with specific package managers:

```bash
# Using uv
sam plugin install my-plugin --install-command "uv pip install {package}"

# Using poetry
sam plugin install my-plugin --install-command "poetry add {package}"
```

Or set environment variable:
```bash
export SAM_PLUGIN_INSTALL_COMMAND="uv pip install {package}"
sam plugin install my-plugin
```

## Using Installed Plugins

### Add Plugin Component

After installing, add a component from the plugin:

```bash
sam plugin add my-slack-bot --plugin sam-slack-gateway
```

This creates a configuration file in `configs/` using the plugin's template.

### Configure the Component

Edit the generated configuration:

```yaml
# configs/gateways/my_slack_bot.yaml
component_config:
  gateway_name: my-slack-bot
  
  # Plugin-specific settings
  slack:
    bot_token: ${SLACK_BOT_TOKEN}
    app_token: ${SLACK_APP_TOKEN}
    signing_secret: ${SLACK_SIGNING_SECRET}
```

### Run with Plugin

```bash
sam run
# Plugin components start automatically
```

## Creating Plugins

### Step 1: Initialize Plugin Project

```bash
sam plugin create my-awesome-plugin --type agent
```

Options:
- `--type agent` - Agent plugin
- `--type gateway` - Gateway plugin
- `--type custom` - Custom functionality

Additional options:
```bash
sam plugin create my-plugin \
  --type agent \
  --author-name "Your Name" \
  --author-email "you@example.com" \
  --description "A plugin that does amazing things" \
  --version "0.1.0"
```

### Step 2: Plugin Project Structure

```
my-awesome-plugin/
├── pyproject.toml           # Package configuration
├── README.md                # Documentation
├── src/
│   └── my_awesome_plugin/
│       ├── __init__.py
│       ├── agent.py         # Agent implementation
│       └── config/
│           └── template.yaml # Configuration template
└── tests/
    └── test_agent.py
```

### Step 3: Implement the Plugin

#### Agent Plugin Example

```python
# src/my_awesome_plugin/agent.py
"""My awesome agent plugin."""

from solace_agent_mesh.agent import BaseAgent
from typing import Dict, Any

class MyAwesomeAgent(BaseAgent):
    """An agent that does awesome things."""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.custom_setting = config.get("custom_setting", "default")
    
    async def process(self, message: str, context: Dict[str, Any]) -> str:
        """Process incoming messages."""
        # Your agent logic here
        return f"Processed: {message}"
```

#### Configuration Template

```yaml
# src/my_awesome_plugin/config/template.yaml
---
# My Awesome Plugin Configuration
# This template is used when running: sam plugin add <name> --plugin my-awesome-plugin

shared_config_file: ${PWD}/configs/shared_config.yaml

flows:
  - name: ${COMPONENT_NAME}-flow
    components:
      - component_name: ${COMPONENT_NAME}
        component_module: my_awesome_plugin.agent
        component_config:
          agent_name: ${COMPONENT_NAME}
          
          agent_instruction: |
            You are an awesome agent.
            # Customize this instruction for your use case
          
          # Plugin-specific settings
          custom_setting: ${CUSTOM_SETTING:default_value}
          
          # Standard agent settings
          model_type: ${MODEL_TYPE:general}
          supports_streaming: true
          
          agent_card:
            description: "My awesome agent that does amazing things"
            defaultInputModes: ["text"]
            defaultOutputModes: ["text"]
            skills:
              - id: awesome_skill
                name: Awesome Skill
                description: Does awesome things
```

#### Package Configuration

```toml
# pyproject.toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "sam-my-awesome-plugin"
version = "0.1.0"
description = "An awesome plugin for Solace Agent Mesh"
readme = "README.md"
requires-python = ">=3.10"
authors = [
    { name = "Your Name", email = "you@example.com" }
]
dependencies = [
    "solace-agent-mesh>=1.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-asyncio>=0.21",
]

[project.entry-points."solace_agent_mesh.plugins"]
my-awesome-plugin = "my_awesome_plugin:plugin_config"

[tool.hatch.build.targets.wheel]
packages = ["src/my_awesome_plugin"]
```

#### Plugin Entry Point

```python
# src/my_awesome_plugin/__init__.py
"""My Awesome Plugin for Solace Agent Mesh."""

from pathlib import Path

plugin_config = {
    "name": "my-awesome-plugin",
    "version": "0.1.0",
    "type": "agent",
    "description": "An awesome plugin that does amazing things",
    "config_template": Path(__file__).parent / "config" / "template.yaml",
    "component_class": "my_awesome_plugin.agent.MyAwesomeAgent",
}

__all__ = ["plugin_config"]
```

### Step 4: Build the Plugin

```bash
cd my-awesome-plugin
sam plugin build
```

This creates:
- `dist/sam_my_awesome_plugin-0.1.0-py3-none-any.whl`
- `dist/sam_my_awesome_plugin-0.1.0.tar.gz`

### Step 5: Test Locally

```bash
# Install locally
sam plugin install ./dist/sam_my_awesome_plugin-0.1.0-py3-none-any.whl

# Add component
sam plugin add test-agent --plugin my-awesome-plugin

# Run and test
sam run
```

## Plugin with Multiple Components

Create plugins with multiple agents or gateways:

### Step 1: Create Base Plugin

```bash
sam plugin create multi-agent-plugin --type agent --skip
```

### Step 2: Add Additional Agents

```bash
cd multi-agent-plugin
sam plugin add data-agent --plugin multi-agent-plugin
sam plugin add report-agent --plugin multi-agent-plugin
```

### Step 3: Structure

```
multi-agent-plugin/
├── pyproject.toml
├── src/
│   └── multi_agent_plugin/
│       ├── __init__.py
│       ├── data_agent.py
│       ├── report_agent.py
│       └── config/
│           ├── data_agent_template.yaml
│           └── report_agent_template.yaml
```

### Step 4: Multi-Component Entry Point

```python
# src/multi_agent_plugin/__init__.py
from pathlib import Path

plugin_config = {
    "name": "multi-agent-plugin",
    "version": "0.1.0",
    "type": "agent",
    "description": "Multiple specialized agents",
    "components": {
        "data-agent": {
            "config_template": Path(__file__).parent / "config" / "data_agent_template.yaml",
            "component_class": "multi_agent_plugin.data_agent.DataAgent",
        },
        "report-agent": {
            "config_template": Path(__file__).parent / "config" / "report_agent_template.yaml",
            "component_class": "multi_agent_plugin.report_agent.ReportAgent",
        },
    },
}
```

## Gateway Plugin Example

### Plugin Structure

```
my-gateway-plugin/
├── pyproject.toml
├── src/
│   └── my_gateway_plugin/
│       ├── __init__.py
│       ├── gateway.py
│       └── config/
│           └── template.yaml
```

### Gateway Implementation

```python
# src/my_gateway_plugin/gateway.py
"""Custom gateway plugin."""

from solace_agent_mesh.gateway import BaseGateway
from typing import Dict, Any
import aiohttp

class MyCustomGateway(BaseGateway):
    """Gateway for custom protocol."""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.webhook_url = config.get("webhook_url")
        self.api_key = config.get("api_key")
    
    async def start(self):
        """Start the gateway."""
        # Initialize connections, start servers, etc.
        self.logger.info("Custom gateway started")
    
    async def stop(self):
        """Stop the gateway."""
        # Cleanup
        self.logger.info("Custom gateway stopped")
    
    async def handle_external_message(self, payload: Dict[str, Any]):
        """Handle messages from external system."""
        # Transform to SAM format
        message = payload.get("text", "")
        metadata = {"source": "custom_gateway", **payload}
        
        # Send to mesh
        response = await self.send_to_mesh(message, metadata)
        
        # Send response back to external system
        await self._send_response(response)
    
    async def _send_response(self, response: str):
        """Send response to external system."""
        async with aiohttp.ClientSession() as session:
            await session.post(
                self.webhook_url,
                json={"text": response},
                headers={"Authorization": f"Bearer {self.api_key}"}
            )
```

## Publishing Plugins

### To PyPI (Public)

```bash
# Build
sam plugin build

# Upload to PyPI
pip install twine
twine upload dist/*
```

### To Private Repository

```bash
# Upload to private PyPI
twine upload --repository-url https://pypi.mycompany.com/simple/ dist/*

# Or use environment variable
export TWINE_REPOSITORY_URL=https://pypi.mycompany.com/simple/
twine upload dist/*
```

### To GitHub Releases

```bash
# Tag and release
git tag v0.1.0
git push origin v0.1.0

# Upload wheel to release
gh release create v0.1.0 dist/*.whl --notes "Initial release"
```

## Plugin Best Practices

### 1. Clear Documentation

Include comprehensive README:

```markdown
# My Awesome Plugin

## Installation

\`\`\`bash
sam plugin install sam-my-awesome-plugin
\`\`\`

## Usage

\`\`\`bash
sam plugin add my-agent --plugin my-awesome-plugin
\`\`\`

## Configuration

| Setting | Description | Default |
|---------|-------------|---------|
| custom_setting | Does X | default |

## Example

\`\`\`yaml
component_config:
  custom_setting: my_value
\`\`\`
```

### 2. Sensible Defaults

```yaml
# template.yaml
component_config:
  # Required - no default
  agent_name: ${COMPONENT_NAME}
  
  # Optional with sensible defaults
  timeout: ${TIMEOUT:60}
  max_retries: ${MAX_RETRIES:3}
  log_level: ${LOG_LEVEL:INFO}
```

### 3. Environment Variable Support

```yaml
component_config:
  # Support env vars for secrets
  api_key: ${MY_PLUGIN_API_KEY}
  
  # Support env vars with defaults for config
  endpoint: ${MY_PLUGIN_ENDPOINT:https://api.default.com}
```

### 4. Comprehensive Testing

```python
# tests/test_agent.py
import pytest
from my_awesome_plugin.agent import MyAwesomeAgent

@pytest.fixture
def agent():
    config = {
        "agent_name": "test-agent",
        "custom_setting": "test_value"
    }
    return MyAwesomeAgent(config)

def test_process(agent):
    result = agent.process("test message", {})
    assert "Processed" in result

@pytest.mark.asyncio
async def test_async_process(agent):
    result = await agent.async_process("test", {})
    assert result is not None
```

### 5. Version Compatibility

```toml
# pyproject.toml
[project]
dependencies = [
    "solace-agent-mesh>=1.0.0,<2.0.0",  # Pin to major version
]
```

## Troubleshooting

### Plugin Not Found After Install

```bash
# Verify installation
pip show sam-my-plugin

# Check entry points
python -c "from importlib.metadata import entry_points; print(entry_points(group='solace_agent_mesh.plugins'))"
```

### Template Not Generating

1. Check template path in plugin_config
2. Verify template YAML syntax
3. Check variable substitution: `${VAR}` format

### Import Errors

```bash
# Test import directly
python -c "from my_plugin.agent import MyAgent"

# Check dependencies
pip install -e .[dev]
```

### Build Failures

```bash
# Check pyproject.toml syntax
python -c "import tomllib; tomllib.load(open('pyproject.toml', 'rb'))"

# Build with verbose output
sam plugin build --verbose
```

## Related Skills

- [create-and-manage-agents](create-and-manage-agents-skill.md) - Agent development
- [create-and-manage-gateways](create-and-manage-gateways-skill.md) - Gateway development
- [configure-agent-tools](configure-agent-tools-skill.md) - Tool integration
