---
name: create-and-manage-agents
description: Create, configure, and manage AI agents in Solace Agent Mesh. Use this skill when adding new agents to your mesh, configuring agent behavior, setting up agent cards, or implementing agent-to-agent communication patterns.
compatibility: Requires initialized SAM project
metadata:
  author: solace
  version: "1.0"
  category: development
  complexity: intermediate
  estimated_time: 30-60min
---

# Create and Manage Agents

This skill covers how to create, configure, and manage AI agents in Solace Agent Mesh. Agents are the intelligent workers of your system, each with specialized capabilities and the ability to collaborate through the A2A (Agent-to-Agent) protocol.

## Prerequisites

- Initialized SAM project (`sam init` completed)
- Understanding of YAML configuration
- Familiarity with LLM prompting concepts

## Understanding SAM Agents

### What is an Agent?

An agent in SAM is:
- An AI-powered component built on Google's Agent Development Kit (ADK)
- Equipped with tools to perform specific tasks
- Discoverable by other agents through the A2A protocol
- Capable of delegating tasks to peer agents

### Agent Architecture

```
┌─────────────────────────────────────────┐
│                 Agent                    │
├─────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────────┐ │
│  │   LLM       │    │  Agent Card     │ │
│  │  (Reasoning)│    │  (Discovery)    │ │
│  └─────────────┘    └─────────────────┘ │
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐│
│  │            Tools                     ││
│  │  ┌────────┐ ┌────────┐ ┌──────────┐││
│  │  │Built-in│ │ Custom │ │   MCP    │││
│  │  └────────┘ └────────┘ └──────────┘││
│  └─────────────────────────────────────┘│
├─────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐│
│  │     A2A Protocol (Communication)    ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

## Creating an Agent

### Method 1: GUI Mode (Recommended)

Launch the browser-based agent configuration:

```bash
sam add agent my-agent --gui
```

This opens an interface where you can configure:
- Agent name and description
- LLM model selection
- Tool assignments
- Discovery settings
- Agent card details

### Method 2: CLI Mode

Create an agent with default settings:

```bash
sam add agent my-agent --skip
```

This creates `configs/agents/my_agent.yaml` with a template configuration.

### Method 3: CLI with Options

Specify options directly:

```bash
sam add agent my-agent \
  --namespace "myorg/dev" \
  --model-type planning \
  --supports-streaming true \
  --agent-card-description "An agent that processes data" \
  --enable-builtin-artifact-tools true
```

## Agent Configuration File

### Complete Agent YAML Structure

```yaml
---
# configs/agents/my_agent.yaml

# Reference to shared configuration
shared_config_file: ${PWD}/configs/shared_config.yaml

# Solace AI Connector configuration
flows:
  - name: my-agent-flow
    components:
      # Agent component configuration
      - component_name: my_agent
        component_module: solace_agent_mesh.agent
        component_config:
          # =========================
          # BASIC SETTINGS
          # =========================
          agent_name: my-agent
          
          # Agent instructions (system prompt)
          agent_instruction: |
            You are a specialized data analysis agent.
            
            Your capabilities:
            - Query databases and analyze results
            - Generate visualizations from data
            - Provide insights and recommendations
            
            Guidelines:
            - Always explain your analysis methodology
            - Cite data sources in your responses
            - Ask for clarification if the request is ambiguous
          
          # =========================
          # LLM CONFIGURATION
          # =========================
          # Model type (references shared_config)
          model_type: planning  # Options: planning, general, multimodal
          
          # Enable streaming responses
          supports_streaming: true
          
          # =========================
          # TOOL CONFIGURATION
          # =========================
          tools:
            # Built-in tools
            builtin:
              artifact_management: true
              data_analysis: true
              web_request: false
              
            # Custom Python tools
            custom:
              - module_path: custom_components.my_tools
                class_name: DatabaseQueryTool
                config:
                  connection_string: ${DATABASE_URL}
          
          # =========================
          # SESSION & ARTIFACTS
          # =========================
          session_service:
            type: memory
            behavior: PERSISTENT
            
          artifact_service:
            type: filesystem
            base_path: ./artifacts
            scope: namespace
            
          # Artifact handling
          artifact_handling_mode: embed  # Options: ignore, embed, reference
          enable_embed_resolution: true
          enable_artifact_content_instruction: true
          
          # =========================
          # AGENT CARD (Discovery)
          # =========================
          agent_card:
            description: |
              Data analysis agent with SQL capabilities.
              Can query databases, analyze data, and generate visualizations.
            defaultInputModes:
              - text
              - application/json
            defaultOutputModes:
              - text
              - file
            skills:
              - id: sql_query
                name: SQL Query Execution
                description: Execute SQL queries on connected databases
              - id: data_visualization
                name: Data Visualization
                description: Generate charts and graphs from data
          
          # =========================
          # DISCOVERY SETTINGS
          # =========================
          agent_discovery:
            enabled: true
            publishing_interval: 30  # seconds
          
          # =========================
          # INTER-AGENT COMMUNICATION
          # =========================
          inter_agent_communication:
            allow_list:
              - "*"  # Allow all agents
            # deny_list:
            #   - restricted-agent
            timeout: 60  # seconds
```

## Agent Instructions (System Prompt)

The `agent_instruction` field defines the agent's behavior. Write effective instructions:

### Instruction Best Practices

```yaml
agent_instruction: |
  # Role Definition
  You are a [specific role] agent specializing in [domain].
  
  # Capabilities
  Your capabilities include:
  - [Capability 1]
  - [Capability 2]
  - [Capability 3]
  
  # Guidelines
  When responding:
  - [Guideline 1]
  - [Guideline 2]
  
  # Constraints
  You should NOT:
  - [Constraint 1]
  - [Constraint 2]
  
  # Output Format
  Structure your responses as:
  1. [Section 1]
  2. [Section 2]
```

### Example: Research Agent

```yaml
agent_instruction: |
  You are a research agent that helps users find and analyze information.
  
  Capabilities:
  - Search the web for information
  - Summarize findings from multiple sources
  - Extract key facts and statistics
  - Cite sources properly
  
  When researching:
  - Use multiple sources to verify information
  - Prioritize recent and authoritative sources
  - Clearly distinguish facts from opinions
  - Always provide citations
  
  Response format:
  1. Summary of findings
  2. Key points with citations
  3. Recommendations for further research
```

### Example: Code Assistant Agent

```yaml
agent_instruction: |
  You are a code assistant agent specializing in Python development.
  
  Capabilities:
  - Write and explain Python code
  - Debug code issues
  - Suggest code improvements
  - Generate documentation
  
  Guidelines:
  - Write clean, readable code following PEP 8
  - Include docstrings and comments
  - Handle edge cases and errors
  - Prefer standard library solutions when possible
  
  When asked to write code:
  1. Understand the requirements
  2. Plan the implementation
  3. Write the code with explanations
  4. Suggest tests if appropriate
```

## Agent Card Configuration

The Agent Card describes your agent's capabilities for discovery:

```yaml
agent_card:
  # Description for other agents and orchestrator
  description: |
    Concise description of what this agent does.
    Include key capabilities and use cases.
  
  # Supported input types
  defaultInputModes:
    - text           # Plain text input
    - application/json  # Structured JSON
    - file           # File artifacts
  
  # Supported output types  
  defaultOutputModes:
    - text           # Plain text responses
    - file           # File artifacts
    - application/json
  
  # Specific skills this agent provides
  skills:
    - id: skill_identifier
      name: Human Readable Name
      description: |
        Detailed description of what this skill does.
        Include parameters it expects and outputs it produces.
```

### Skills Best Practices

Each skill should map to a specific capability:

```yaml
skills:
  # Good: Specific, actionable skills
  - id: generate_report
    name: Generate Analytics Report
    description: |
      Generates a comprehensive analytics report from database data.
      Input: date_range, metrics list
      Output: PDF report artifact
      
  - id: query_database
    name: Execute SQL Query
    description: |
      Executes read-only SQL queries against the connected database.
      Input: SQL query string
      Output: Query results as JSON
      
  # Avoid: Vague, broad skills
  # - id: help
  #   name: Help with stuff
  #   description: Does various things
```

## Inter-Agent Communication

### Allow/Deny Lists

Control which agents can communicate:

```yaml
inter_agent_communication:
  # Allow specific agents
  allow_list:
    - orchestrator
    - data-agent
    - report-agent
    
  # Or allow all
  allow_list:
    - "*"
    
  # Block specific agents
  deny_list:
    - untrusted-agent
    
  # Communication timeout
  timeout: 60
```

### Delegation Patterns

Agents can delegate tasks to peers. The orchestrator automatically routes requests to appropriate agents based on their Agent Cards.

**Hub-and-Spoke Pattern:**
```
          ┌─────────────────┐
          │   Orchestrator  │
          └────────┬────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
┌───────┐    ┌───────────┐   ┌─────────┐
│Agent A│    │  Agent B  │   │ Agent C │
└───────┘    └───────────┘   └─────────┘
```

**Peer-to-Peer Pattern:**
```
┌───────┐ ◄──► ┌───────────┐
│Agent A│      │  Agent B  │
└───────┘      └─────┬─────┘
                     │
                     ▼
               ┌─────────┐
               │ Agent C │
               └─────────┘
```

## Model Type Selection

Choose the appropriate model for your agent's tasks:

```yaml
# For complex reasoning and planning
model_type: planning
# Uses: LLM_SERVICE_PLANNING_MODEL_NAME (e.g., gpt-4o)

# For simple tasks and quick responses
model_type: general
# Uses: LLM_SERVICE_GENERAL_MODEL_NAME (e.g., gpt-4o-mini)

# For image understanding
model_type: multimodal
# Requires vision-capable model
```

## Session Management

Configure how the agent maintains conversation context:

```yaml
session_service:
  # Storage type
  type: memory       # In-memory (development)
  # type: vertex_rag  # Persistent (production)
  
  # Session behavior
  behavior: PERSISTENT   # Keep across restarts
  # behavior: RUN_BASED  # Clear on restart
```

## Artifact Handling

Control how agents work with files:

```yaml
artifact_service:
  type: filesystem
  base_path: ./artifacts
  scope: namespace  # namespace, app, custom

# How to handle artifacts in prompts
artifact_handling_mode: embed     # Include content in prompt
# artifact_handling_mode: reference  # Reference by ID
# artifact_handling_mode: ignore    # Don't include

# Additional artifact settings
enable_embed_resolution: true
enable_artifact_content_instruction: true
enable_builtin_artifact_tools: true
```

## Common Agent Patterns

### 1. Data Processing Agent

```yaml
agent_name: data-processor
agent_instruction: |
  You process and transform data files.
  Accept CSV, JSON, and Excel files.
  Output cleaned, transformed data.
  
tools:
  builtin:
    artifact_management: true
    data_analysis: true
    
agent_card:
  description: "Data processing agent for file transformations"
  defaultInputModes: [file, text]
  defaultOutputModes: [file, text]
  skills:
    - id: transform_csv
      name: Transform CSV
      description: Clean and transform CSV files
```

### 2. Research Agent

```yaml
agent_name: researcher
agent_instruction: |
  You research topics using web search.
  Synthesize information from multiple sources.
  Always cite your sources.
  
tools:
  builtin:
    web_request: true
    
agent_card:
  description: "Research agent with web search capabilities"
  defaultInputModes: [text]
  defaultOutputModes: [text]
  skills:
    - id: web_research
      name: Web Research
      description: Search and synthesize information from the web
```

### 3. Integration Agent

```yaml
agent_name: api-integrator
agent_instruction: |
  You integrate with external APIs.
  Handle authentication and error responses.
  Transform data between formats.
  
tools:
  custom:
    - module_path: custom_components.api_tools
      class_name: ExternalAPITool
      
agent_card:
  description: "API integration agent"
  defaultInputModes: [application/json, text]
  defaultOutputModes: [application/json, text]
```

## Testing Your Agent

### 1. Run in Isolation

```bash
# Run just your agent with shared config
sam run configs/agents/my_agent.yaml
```

### 2. Enable Debug Logging

```bash
LOG_LEVEL=DEBUG sam run configs/agents/my_agent.yaml
```

### 3. Check Agent Discovery

Look for discovery messages in logs:
```
INFO - Agent 'my-agent' publishing agent card
DEBUG - Agent card published: {...}
```

### 4. Test via Web UI

```bash
# Run full mesh with Web UI
sam run

# Open http://localhost:8000
# Send test messages to your agent
```

## Troubleshooting

### Agent Not Discovered

1. Check discovery is enabled:
   ```yaml
   agent_discovery:
     enabled: true
   ```

2. Verify agent is running:
   ```bash
   LOG_LEVEL=DEBUG sam run
   # Look for: "Agent 'my-agent' initialized"
   ```

3. Check broker connection

### Agent Not Responding

1. Check agent logs for errors
2. Verify LLM configuration
3. Test LLM connection directly
4. Check tool execution errors

### Tools Not Working

1. Verify tool configuration:
   ```yaml
   tools:
     builtin:
       artifact_management: true  # Not just listed
   ```

2. Check custom tool imports:
   ```bash
   python -c "from custom_components.my_tools import MyTool"
   ```

### Memory Issues

For agents processing large data:
```yaml
# Use reference mode instead of embed
artifact_handling_mode: reference

# Or use streaming
supports_streaming: true
```

## Related Skills

- [configure-agent-tools](configure-agent-tools-skill.md) - Tool configuration details
- [configure-shared-settings](configure-shared-settings-skill.md) - Shared configuration
- [run-and-debug-projects](run-and-debug-projects-skill.md) - Running and debugging
