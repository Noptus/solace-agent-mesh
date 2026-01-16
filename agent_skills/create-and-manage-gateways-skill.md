---
name: create-and-manage-gateways
description: Create and configure gateways to expose Solace Agent Mesh to external systems. Use this skill when building interfaces for REST APIs, web UIs, webhooks, Slack, or custom integrations that connect external users and systems to your agent mesh.
compatibility: Requires initialized SAM project
metadata:
  author: solace
  version: "1.0"
  category: development
  complexity: intermediate
  estimated_time: 30-45min
---

# Create and Manage Gateways

This skill covers how to create and configure gateways in Solace Agent Mesh. Gateways are the entry points that connect external systems, users, and applications to your agent mesh through various protocols.

## Prerequisites

- Initialized SAM project (`sam init` completed)
- Understanding of the protocol/interface you want to expose (REST, webhooks, etc.)
- For custom gateways: Python development knowledge

## Understanding Gateways

### What is a Gateway?

A gateway in SAM:
- Acts as the entry point from external systems to the agent mesh
- Translates external requests into A2A protocol messages
- Handles authentication, authorization, and user enrichment
- Formats responses for the external system

### Gateway Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Gateway                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   External   │    │   Protocol   │    │    A2A       │  │
│  │   Interface  │───▶│  Translation │───▶│   Message    │  │
│  │  (REST/WS)   │    │              │    │   Router     │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │    Auth &    │    │   System     │    │   Response   │  │
│  │   Identity   │    │   Purpose    │    │  Formatting  │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │   Agent Mesh    │
                    │   (via Broker)  │
                    └─────────────────┘
```

### Available Gateway Types

| Type | Use Case | Protocol |
|------|----------|----------|
| HTTP SSE | Web UI with streaming | HTTP + Server-Sent Events |
| REST | API integration | HTTP REST |
| Webhook | Incoming event handlers | HTTP webhooks |
| Event Mesh | Broker-to-broker | Solace PubSub+ |
| Slack | Team chat | Slack API |
| Custom | Any protocol | Python implementation |

## Creating a Gateway

### Method 1: GUI Mode (Recommended)

Launch the browser-based configuration:

```bash
sam add gateway my-gateway --gui
```

Configure:
- Gateway name and ID
- System purpose (context for requests)
- Response formatting rules
- Interface type and settings
- Authentication settings

### Method 2: CLI Mode

Create with default settings:

```bash
sam add gateway my-gateway --skip
```

This creates `configs/gateways/my_gateway.yaml` with a template configuration.

### Method 3: CLI with Options

Specify options directly:

```bash
sam add gateway my-gateway \
  --namespace "myorg/prod" \
  --gateway-id "gw-001" \
  --system-purpose "You are a helpful assistant for customer support" \
  --response-format "Respond in clear, friendly language"
```

## Gateway Configuration File

### Complete Gateway YAML Structure

```yaml
---
# configs/gateways/my_gateway.yaml

# Reference shared configuration
shared_config_file: ${PWD}/configs/shared_config.yaml

# Gateway flow configuration
flows:
  - name: my-gateway-flow
    components:
      - component_name: my_gateway
        component_module: solace_agent_mesh.gateway
        component_config:
          # =========================
          # BASIC SETTINGS
          # =========================
          gateway_name: my-gateway
          gateway_id: ${GATEWAY_ID:gw-001}
          
          # =========================
          # SYSTEM PURPOSE
          # =========================
          # Context provided to agents for requests through this gateway
          system_purpose: |
            You are a helpful AI assistant accessed through our API.
            
            Context:
            - Users are developers integrating with our platform
            - Provide technical, accurate responses
            - Include code examples when relevant
          
          # =========================
          # RESPONSE FORMATTING
          # =========================
          # Instructions for formatting responses
          response_format: |
            Format your responses as follows:
            - Use markdown for code blocks
            - Include section headers for long responses
            - Be concise but thorough
          
          # =========================
          # INTERFACE CONFIGURATION
          # =========================
          interface:
            type: http_sse  # http_sse, rest, webhook
            host: ${GATEWAY_HOST:0.0.0.0}
            port: ${GATEWAY_PORT:8000}
            
            # SSL/TLS settings (optional)
            # ssl:
            #   enabled: true
            #   cert_file: /path/to/cert.pem
            #   key_file: /path/to/key.pem
          
          # =========================
          # AUTHENTICATION
          # =========================
          authentication:
            enabled: false
            # type: api_key
            # api_key_header: X-API-Key
            # api_keys:
            #   - ${API_KEY_1}
            #   - ${API_KEY_2}
          
          # =========================
          # ARTIFACT HANDLING
          # =========================
          artifact_service:
            type: filesystem
            base_path: ./gateway_artifacts
            scope: namespace
```

## HTTP SSE Gateway (Web UI)

The most common gateway for interactive web interfaces with streaming responses.

### Configuration

```yaml
# configs/gateways/webui_gateway.yaml
component_config:
  gateway_name: webui-gateway
  gateway_id: webui-gw
  
  system_purpose: |
    You are a helpful AI assistant in a web chat interface.
    Be conversational and friendly.
    Format responses with markdown for readability.
  
  response_format: |
    Use markdown formatting:
    - **Bold** for emphasis
    - `code` for technical terms
    - Code blocks for code snippets
    - Bullet points for lists
  
  interface:
    type: http_sse
    host: ${FASTAPI_HOST:0.0.0.0}
    port: ${FASTAPI_PORT:8000}
    
    # SSE-specific settings
    sse:
      heartbeat_interval: 30  # seconds
      timeout: 300  # max request duration
    
    # CORS settings
    cors:
      enabled: true
      allow_origins:
        - "http://localhost:3000"
        - "https://myapp.com"
      allow_methods: ["GET", "POST", "OPTIONS"]
      allow_headers: ["*"]
  
  # Web UI frontend settings
  frontend:
    welcome_message: "Hello! How can I help you today?"
    bot_name: "AI Assistant"
    logo_url: "/static/logo.png"
    collect_feedback: true
```

### Endpoints

The HTTP SSE gateway provides:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/chat` | POST | Send message, receive SSE stream |
| `/agents` | GET | List available agents |
| `/upload` | POST | Upload files/artifacts |
| `/download/{id}` | GET | Download artifacts |
| `/health` | GET | Health check |

### Usage Example

```bash
# Send a message and receive streaming response
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello, how are you?"}' \
  --no-buffer
```

## REST Gateway

For programmatic API access with request/response pattern.

### Configuration

```yaml
# configs/gateways/rest_gateway.yaml
component_config:
  gateway_name: rest-gateway
  gateway_id: rest-gw
  
  system_purpose: |
    You are an API backend assistant.
    Provide structured, machine-parseable responses.
    Return JSON when appropriate.
  
  response_format: |
    Return responses in JSON format when data is involved:
    {
      "status": "success",
      "data": {...},
      "message": "Human readable message"
    }
  
  interface:
    type: rest
    host: ${REST_HOST:0.0.0.0}
    port: ${REST_PORT:8001}
    
    # REST-specific settings
    rest:
      timeout: 120  # seconds
      max_request_size: 10485760  # 10MB
  
  authentication:
    enabled: true
    type: api_key
    api_key_header: X-API-Key
    api_keys:
      - ${REST_API_KEY}
```

### Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/task` | POST | Submit task, returns task_id |
| `/task/{id}` | GET | Get task status/result |
| `/task/{id}` | DELETE | Cancel task |

### Usage Example

```bash
# Submit a task
TASK_ID=$(curl -s -X POST http://localhost:8001/task \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"message": "Analyze this data"}' | jq -r '.task_id')

# Poll for result
curl http://localhost:8001/task/$TASK_ID \
  -H "X-API-Key: your-api-key"
```

## Webhook Gateway

For receiving events from external systems (GitHub, Stripe, etc.).

### Configuration

```yaml
# configs/gateways/webhook_gateway.yaml
component_config:
  gateway_name: webhook-gateway
  gateway_id: webhook-gw
  
  system_purpose: |
    You process incoming webhook events.
    Extract relevant information and take appropriate actions.
  
  interface:
    type: webhook
    host: ${WEBHOOK_HOST:0.0.0.0}
    port: ${WEBHOOK_PORT:8002}
    
    webhook:
      # Path for webhook endpoint
      path: /webhook
      
      # Signature verification (optional)
      verify_signature: true
      signature_header: X-Hub-Signature-256
      secret: ${WEBHOOK_SECRET}
      
      # Transform incoming payload
      payload_mapping:
        message: "$.event.description"
        metadata: "$.event"
```

### Usage Example

```bash
# GitHub webhook
curl -X POST http://localhost:8002/webhook \
  -H "Content-Type: application/json" \
  -H "X-Hub-Signature-256: sha256=..." \
  -d '{"event": "push", "repository": "myrepo"}'
```

## Event Mesh Gateway (Plugin)

Connect to external Solace brokers for system-to-system integration.

### Installation

```bash
sam plugin install sam-event-mesh-gateway
sam plugin add event-gateway --plugin sam-event-mesh-gateway
```

### Configuration

```yaml
# configs/gateways/event_mesh_gateway.yaml
component_config:
  gateway_name: event-mesh-gateway
  gateway_id: event-gw
  
  system_purpose: |
    You process events from the enterprise event mesh.
    Handle events according to business rules.
  
  interface:
    type: event_mesh
    
    # External broker connection
    broker:
      url: ${EXTERNAL_BROKER_URL}
      vpn: ${EXTERNAL_BROKER_VPN}
      username: ${EXTERNAL_BROKER_USER}
      password: ${EXTERNAL_BROKER_PASS}
    
    # Subscription topics
    subscriptions:
      - topic: "events/orders/*"
        transform: order_event
      - topic: "events/customers/*"
        transform: customer_event
```

## Slack Gateway (Plugin)

Integrate with Slack for team chat interactions.

### Installation

```bash
sam plugin install sam-slack-gateway
sam plugin add slack-gateway --plugin sam-slack-gateway
```

### Configuration

```yaml
# configs/gateways/slack_gateway.yaml
component_config:
  gateway_name: slack-gateway
  gateway_id: slack-gw
  
  system_purpose: |
    You are a helpful Slack bot for the engineering team.
    Be concise and use Slack formatting (mrkdwn).
  
  response_format: |
    Use Slack mrkdwn formatting:
    - *bold* for emphasis
    - `code` for technical terms
    - ```code blocks``` for code
    - :emoji: where appropriate
  
  interface:
    type: slack
    
    slack:
      bot_token: ${SLACK_BOT_TOKEN}
      app_token: ${SLACK_APP_TOKEN}
      signing_secret: ${SLACK_SIGNING_SECRET}
      
      # Respond in threads
      use_threads: true
      
      # Channels to monitor (optional, bot responds to @mentions by default)
      channels:
        - general
        - engineering
```

## Custom Gateway Development

Create custom gateways for specialized protocols.

### Step 1: Create Gateway Template

```bash
sam add gateway my-custom-gateway
```

This creates a Python template file.

### Step 2: Implement Gateway Class

```python
# custom_components/my_custom_gateway.py
"""Custom gateway implementation."""

from solace_agent_mesh.gateway import BaseGateway
from typing import Dict, Any
import asyncio

class MyCustomGateway(BaseGateway):
    """Custom gateway for proprietary protocol."""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.host = config.get("host", "0.0.0.0")
        self.port = config.get("port", 9000)
        self._server = None
    
    async def start(self):
        """Start the gateway server."""
        self._server = await asyncio.start_server(
            self._handle_connection,
            self.host,
            self.port
        )
        self.logger.info(f"Custom gateway listening on {self.host}:{self.port}")
    
    async def stop(self):
        """Stop the gateway server."""
        if self._server:
            self._server.close()
            await self._server.wait_closed()
    
    async def _handle_connection(self, reader, writer):
        """Handle incoming connections."""
        try:
            data = await reader.read(4096)
            message = self._parse_message(data)
            
            # Send to agent mesh
            response = await self.send_to_mesh(
                message=message["content"],
                metadata=message.get("metadata", {})
            )
            
            # Send response back
            writer.write(self._format_response(response))
            await writer.drain()
        finally:
            writer.close()
    
    def _parse_message(self, data: bytes) -> Dict[str, Any]:
        """Parse incoming message."""
        # Implement your protocol parsing
        return {"content": data.decode(), "metadata": {}}
    
    def _format_response(self, response: str) -> bytes:
        """Format response for your protocol."""
        return response.encode()
```

### Step 3: Configure Gateway

```yaml
# configs/gateways/my_custom_gateway.yaml
component_config:
  gateway_name: my-custom-gateway
  gateway_id: custom-gw
  
  # Use custom gateway class
  gateway_class: custom_components.my_custom_gateway.MyCustomGateway
  
  system_purpose: |
    You handle requests from our proprietary system.
  
  interface:
    host: ${CUSTOM_HOST:0.0.0.0}
    port: ${CUSTOM_PORT:9000}
```

## System Purpose Best Practices

The `system_purpose` provides context for all requests:

### Good System Purpose

```yaml
system_purpose: |
  You are the AI assistant for Acme Corp's developer portal.
  
  Context:
  - Users are software developers using our API
  - They may ask about documentation, code examples, or troubleshooting
  - You have access to our knowledge base and can search documentation
  
  Guidelines:
  - Provide accurate, technical responses
  - Include code examples in relevant programming languages
  - Link to documentation when available
  - Escalate complex issues to human support
  
  You should NOT:
  - Provide information about internal systems
  - Share customer data
  - Make promises about features or timelines
```

### Context-Specific Purposes

Different gateways can have different purposes:

```yaml
# Customer-facing gateway
system_purpose: |
  You are a friendly customer support assistant.
  Be helpful and empathetic.

# Internal API gateway
system_purpose: |
  You are a technical API assistant.
  Provide structured, precise responses.

# Admin gateway
system_purpose: |
  You are an administrative assistant with elevated access.
  Follow security protocols strictly.
```

## Authentication Patterns

### API Key Authentication

```yaml
authentication:
  enabled: true
  type: api_key
  api_key_header: X-API-Key
  api_keys:
    - ${API_KEY_1}
    - ${API_KEY_2}
```

### JWT Authentication

```yaml
authentication:
  enabled: true
  type: jwt
  jwt:
    secret: ${JWT_SECRET}
    algorithm: HS256
    issuer: myapp.com
```

### OAuth 2.0

```yaml
authentication:
  enabled: true
  type: oauth2
  oauth2:
    provider: azure  # google, okta, custom
    client_id: ${OAUTH_CLIENT_ID}
    tenant_id: ${OAUTH_TENANT_ID}
```

## Testing Gateways

### Health Check

```bash
curl http://localhost:8000/health
```

### Test Message

```bash
# HTTP SSE
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}' \
  --no-buffer

# REST
curl -X POST http://localhost:8001/task \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'
```

### Debug Mode

```bash
LOG_LEVEL=DEBUG sam run configs/gateways/my_gateway.yaml
```

## Troubleshooting

### Gateway Won't Start

1. Check port availability:
   ```bash
   lsof -i :8000
   ```

2. Verify configuration:
   ```bash
   python -c "import yaml; yaml.safe_load(open('configs/gateways/my_gateway.yaml'))"
   ```

### Authentication Failing

1. Check API key/token format
2. Verify header name matches configuration
3. Check logs for auth errors:
   ```bash
   LOG_LEVEL=DEBUG sam run
   ```

### No Response from Agents

1. Verify agents are running
2. Check broker connectivity
3. Verify system_purpose is appropriate
4. Check agent discovery

## Related Skills

- [initialize-project](initialize-project-skill.md) - Project setup with gateways
- [create-and-manage-agents](create-and-manage-agents-skill.md) - Agents that gateways connect to
- [manage-plugins](manage-plugins-skill.md) - Gateway plugins (Slack, Event Mesh)
