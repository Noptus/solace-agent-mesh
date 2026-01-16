---
name: configure-agent-tools
description: Configure and integrate tools for Solace Agent Mesh agents including built-in tools, custom Python tools, and MCP (Model Context Protocol) tools. Use this skill when equipping agents with capabilities to interact with external systems, databases, APIs, or custom functionality.
compatibility: Requires initialized SAM project with at least one agent
metadata:
  author: solace
  version: "1.0"
  category: development
  complexity: intermediate
  estimated_time: 30-45min
---

# Configure Agent Tools

This skill covers how to equip Solace Agent Mesh agents with tools - the capabilities that allow agents to perform actions beyond simple text generation. Tools enable agents to query databases, call APIs, manage files, and integrate with external systems.

## Prerequisites

- Initialized SAM project with agents configured
- Python knowledge (for custom tools)
- Understanding of the tool you want to integrate

## Understanding Tools in SAM

### What are Tools?

Tools are callable functions that agents can invoke to:
- Access external data (databases, APIs, files)
- Perform actions (send emails, create files, trigger workflows)
- Transform data (process images, analyze documents)
- Integrate with other systems (MCP servers, custom services)

### Tool Types

```
┌─────────────────────────────────────────────────────┐
│                   Agent Tools                        │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │  Built-in    │  │   Custom     │  │    MCP    │ │
│  │   Tools      │  │  Python      │  │   Tools   │ │
│  │              │  │   Tools      │  │           │ │
│  │ • Artifacts  │  │              │  │ • Stdio   │ │
│  │ • Data       │  │ • Functions  │  │ • HTTP    │ │
│  │ • Web        │  │ • Classes    │  │ • Docker  │ │
│  │ • Research   │  │ • Factories  │  │           │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
│                                                      │
└─────────────────────────────────────────────────────┘
```

## Built-in Tools

SAM provides pre-built tools for common tasks. List available tools:

```bash
sam tools list
```

For detailed information:
```bash
sam tools list --detailed
```

Filter by category:
```bash
sam tools list --category artifact_management
sam tools list --category data_analysis
sam tools list --category web
```

### Enabling Built-in Tools

In your agent configuration:

```yaml
# configs/agents/my_agent.yaml
component_config:
  tools:
    builtin:
      artifact_management: true
      data_analysis: true
      web_request: true
      research: false  # Disabled
```

### Available Built-in Tool Categories

#### Artifact Management Tools
Tools for file and artifact handling:

```yaml
tools:
  builtin:
    artifact_management: true
```

Provides:
- `save_artifact` - Save content to an artifact
- `load_artifact` - Load artifact content
- `list_artifacts` - List available artifacts
- `delete_artifact` - Remove an artifact

#### Data Analysis Tools
Tools for data processing and analysis:

```yaml
tools:
  builtin:
    data_analysis: true
```

Provides:
- `analyze_data` - Analyze structured data
- `generate_chart` - Create visualizations
- `execute_python` - Run Python code for analysis

#### Web/Research Tools
Tools for web access and research:

```yaml
tools:
  builtin:
    web_request: true
    research: true
```

Provides:
- `web_search` - Search the web
- `fetch_url` - Retrieve URL content
- `scrape_page` - Extract page content

#### Image Tools
Tools for image processing:

```yaml
tools:
  builtin:
    image_tools: true
```

Provides:
- `generate_image` - Generate images (requires image model)
- `analyze_image` - Analyze image content (requires multimodal model)

## Custom Python Tools

Create custom tools for specialized functionality.

### Simple Function Tool

Create a file `custom_components/my_tools.py`:

```python
"""Custom tools for my agent."""

from google.adk.tools import FunctionTool

def search_database(query: str, limit: int = 10) -> dict:
    """
    Search the database with a natural language query.
    
    Args:
        query: The search query string
        limit: Maximum number of results to return
        
    Returns:
        Dictionary with search results
    """
    # Your implementation here
    results = perform_database_search(query, limit)
    return {
        "status": "success",
        "count": len(results),
        "results": results
    }

# Export as a tool
database_search_tool = FunctionTool(search_database)
```

Configure in agent YAML:

```yaml
tools:
  custom:
    - module_path: custom_components.my_tools
      function_name: search_database
```

### Class-Based Tool

For more complex tools with state or configuration:

```python
"""Custom class-based tool."""

from google.adk.tools import BaseTool
from pydantic import BaseModel, Field

class DatabaseQueryInput(BaseModel):
    """Input schema for database query tool."""
    query: str = Field(description="SQL query to execute")
    database: str = Field(default="main", description="Target database")

class DatabaseQueryTool(BaseTool):
    """Tool for executing database queries."""
    
    name: str = "database_query"
    description: str = "Execute SQL queries against the database"
    
    def __init__(self, connection_string: str):
        super().__init__()
        self.connection_string = connection_string
        self._connection = None
    
    def _connect(self):
        if self._connection is None:
            import sqlite3
            self._connection = sqlite3.connect(self.connection_string)
        return self._connection
    
    async def _arun(self, query: str, database: str = "main") -> dict:
        """Execute the query asynchronously."""
        conn = self._connect()
        cursor = conn.cursor()
        try:
            cursor.execute(query)
            results = cursor.fetchall()
            columns = [desc[0] for desc in cursor.description] if cursor.description else []
            return {
                "status": "success",
                "columns": columns,
                "rows": results,
                "count": len(results)
            }
        except Exception as e:
            return {
                "status": "error",
                "error": str(e)
            }
    
    def _run(self, query: str, database: str = "main") -> dict:
        """Synchronous execution wrapper."""
        import asyncio
        return asyncio.run(self._arun(query, database))
```

Configure with initialization parameters:

```yaml
tools:
  custom:
    - module_path: custom_components.database_tool
      class_name: DatabaseQueryTool
      config:
        connection_string: ${DATABASE_URL}
```

### Tool Factory Pattern

Create multiple tools from a single class:

```python
"""Tool factory for generating multiple related tools."""

from google.adk.tools import FunctionTool
from typing import List

class APIToolFactory:
    """Factory for creating API integration tools."""
    
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        self.api_key = api_key
    
    def create_tools(self) -> List[FunctionTool]:
        """Create and return all API tools."""
        return [
            self._create_get_tool(),
            self._create_post_tool(),
            self._create_list_tool()
        ]
    
    def _create_get_tool(self) -> FunctionTool:
        def get_resource(resource_id: str) -> dict:
            """Get a resource by ID."""
            import requests
            response = requests.get(
                f"{self.base_url}/resources/{resource_id}",
                headers={"Authorization": f"Bearer {self.api_key}"}
            )
            return response.json()
        return FunctionTool(get_resource)
    
    def _create_post_tool(self) -> FunctionTool:
        def create_resource(name: str, data: dict) -> dict:
            """Create a new resource."""
            import requests
            response = requests.post(
                f"{self.base_url}/resources",
                headers={"Authorization": f"Bearer {self.api_key}"},
                json={"name": name, "data": data}
            )
            return response.json()
        return FunctionTool(create_resource)
    
    def _create_list_tool(self) -> FunctionTool:
        def list_resources(page: int = 1, limit: int = 10) -> dict:
            """List all resources with pagination."""
            import requests
            response = requests.get(
                f"{self.base_url}/resources",
                headers={"Authorization": f"Bearer {self.api_key}"},
                params={"page": page, "limit": limit}
            )
            return response.json()
        return FunctionTool(list_resources)
```

Configure factory:

```yaml
tools:
  custom:
    - module_path: custom_components.api_tools
      class_name: APIToolFactory
      factory_method: create_tools
      config:
        base_url: ${API_BASE_URL}
        api_key: ${API_KEY}
```

### Tool Best Practices

#### 1. Clear Docstrings

The LLM uses docstrings to understand tools:

```python
def my_tool(param1: str, param2: int = 10) -> dict:
    """
    Brief description of what this tool does.
    
    Use this tool when you need to [specific use case].
    
    Args:
        param1: Description of param1 and expected format
        param2: Description of param2, defaults to 10
        
    Returns:
        Dictionary containing:
        - status: "success" or "error"
        - result: The actual result data
        - message: Human-readable message
        
    Example:
        my_tool("search term", limit=5)
    """
```

#### 2. Proper Error Handling

Always return structured responses:

```python
def safe_tool(input_data: str) -> dict:
    """Tool with proper error handling."""
    try:
        # Validate input
        if not input_data:
            return {"status": "error", "error": "Input cannot be empty"}
        
        # Process
        result = process(input_data)
        
        return {
            "status": "success",
            "result": result
        }
    except ValidationError as e:
        return {"status": "error", "error": f"Invalid input: {e}"}
    except ConnectionError as e:
        return {"status": "error", "error": f"Connection failed: {e}"}
    except Exception as e:
        return {"status": "error", "error": f"Unexpected error: {e}"}
```

#### 3. Type Hints

Always use type hints for parameters:

```python
from typing import List, Optional, Dict, Any

def typed_tool(
    query: str,
    filters: Optional[Dict[str, Any]] = None,
    limit: int = 10,
    include_metadata: bool = False
) -> Dict[str, Any]:
    """Tool with proper type hints."""
    ...
```

## MCP Tools (Model Context Protocol)

Integrate external MCP servers for additional capabilities.

### MCP Tool Types

1. **Stdio MCP** - Local process communication
2. **HTTP SSE MCP** - Remote HTTP server
3. **Docker MCP** - Containerized MCP servers

### Configuring Stdio MCP

For local MCP servers:

```yaml
tools:
  mcp:
    - type: stdio
      name: filesystem-mcp
      command: npx
      args:
        - "-y"
        - "@modelcontextprotocol/server-filesystem"
        - "/path/to/allowed/directory"
```

### Configuring HTTP SSE MCP

For remote MCP servers:

```yaml
tools:
  mcp:
    - type: http_sse
      name: remote-mcp
      url: http://localhost:3000/mcp
      # Optional authentication
      headers:
        Authorization: "Bearer ${MCP_API_KEY}"
```

### Configuring Docker MCP

For containerized MCP servers:

```yaml
tools:
  mcp:
    - type: docker
      name: database-mcp
      image: myorg/database-mcp:latest
      environment:
        DATABASE_URL: ${DATABASE_URL}
      ports:
        - "3001:3000"
```

### MCP Integration Example

Full agent with MCP tools:

```yaml
# configs/agents/mcp_agent.yaml
component_config:
  agent_name: mcp-enabled-agent
  
  agent_instruction: |
    You have access to filesystem and database tools through MCP.
    Use them to help users manage files and query data.
  
  tools:
    builtin:
      artifact_management: true
      
    mcp:
      # Filesystem access
      - type: stdio
        name: filesystem
        command: npx
        args: ["-y", "@modelcontextprotocol/server-filesystem", "./data"]
        
      # Database access
      - type: http_sse
        name: postgres
        url: http://localhost:3000/mcp
        headers:
          Authorization: "Bearer ${MCP_DB_KEY}"
```

## Tool Configuration Patterns

### Environment-Based Tool Config

```yaml
tools:
  custom:
    - module_path: custom_components.tools
      class_name: MyTool
      config:
        # Use environment variables
        api_key: ${TOOL_API_KEY}
        endpoint: ${TOOL_ENDPOINT:https://api.default.com}
        timeout: ${TOOL_TIMEOUT:30}
```

### Conditional Tools

Enable tools based on environment:

```yaml
tools:
  builtin:
    artifact_management: true
    # Development-only tools
    data_analysis: ${ENABLE_DATA_TOOLS:false}
```

### Tool Scopes

Limit tool access:

```yaml
tools:
  builtin:
    artifact_management:
      enabled: true
      scope: read_only  # read_only, read_write, admin
```

## Testing Tools

### Unit Testing Custom Tools

```python
# tests/test_my_tools.py
import pytest
from custom_components.my_tools import search_database, DatabaseQueryTool

def test_search_database():
    """Test the search function."""
    result = search_database("test query", limit=5)
    assert result["status"] == "success"
    assert "results" in result
    assert len(result["results"]) <= 5

def test_database_tool():
    """Test the database tool class."""
    tool = DatabaseQueryTool(connection_string="sqlite:///test.db")
    result = tool._run("SELECT 1")
    assert result["status"] == "success"

@pytest.fixture
def mock_api():
    """Mock external API for testing."""
    with patch("requests.get") as mock:
        mock.return_value.json.return_value = {"data": []}
        yield mock
```

### Integration Testing

```bash
# Test tools work with agent
LOG_LEVEL=DEBUG sam run configs/agents/my_agent.yaml

# Check tool invocations in logs
# DEBUG - Tool invocation: search_database
# DEBUG - Tool result: {"status": "success", ...}
```

## Troubleshooting

### Tool Not Found

```
ERROR - Tool 'my_tool' not found
```

1. Check module path:
   ```bash
   python -c "from custom_components.my_tools import MyTool"
   ```

2. Verify YAML syntax:
   ```yaml
   tools:
     custom:
       - module_path: custom_components.my_tools  # No .py extension
         class_name: MyTool
   ```

### Tool Execution Error

```
ERROR - Tool execution failed: ImportError
```

1. Check dependencies are installed:
   ```bash
   pip install required-package
   ```

2. Verify tool initialization:
   ```python
   tool = MyTool(config={"key": "value"})
   tool._run("test")
   ```

### MCP Connection Failed

```
ERROR - MCP connection refused
```

1. Check MCP server is running
2. Verify URL/command configuration
3. Test connectivity:
   ```bash
   curl http://localhost:3000/mcp/health
   ```

### Tool Returns Wrong Format

The LLM may misinterpret tool output if:
- Missing or unclear docstrings
- Inconsistent return format
- No error handling

Fix by standardizing output:
```python
def consistent_tool(input: str) -> dict:
    """
    Always returns: {"status": str, "data": any, "message": str}
    """
    return {
        "status": "success",
        "data": result,
        "message": "Operation completed"
    }
```

## Related Skills

- [create-and-manage-agents](create-and-manage-agents-skill.md) - Agent configuration
- [manage-plugins](manage-plugins-skill.md) - Plugin-based tools
- [run-and-debug-projects](run-and-debug-projects-skill.md) - Testing tools
