---
name: run-and-debug-projects
description: Run, monitor, and troubleshoot Solace Agent Mesh applications. Use this skill when executing SAM projects, analyzing logs, debugging agent behavior, or resolving runtime issues.
compatibility: Requires initialized SAM project with configuration files
metadata:
  author: solace
  version: "1.0"
  category: operations
  complexity: beginner
  estimated_time: 15-30min
---

# Run and Debug SAM Projects

This skill covers how to run Solace Agent Mesh applications, monitor their behavior, analyze logs, and troubleshoot common issues.

## Prerequisites

- Initialized SAM project (`sam init` completed)
- Configuration files in `configs/` directory
- Environment variables set in `.env`

## Running the Application

### Basic Run Command

Start your agent mesh with all configured components:

```bash
sam run
```

This command:
1. Loads environment variables from `.env`
2. Discovers all YAML config files in `configs/`
3. Starts all agents, gateways, and services
4. Connects to the message broker

### Run Specific Files

Run only specific configuration files:

```bash
# Run a single agent
sam run configs/agents/my_agent.yaml

# Run multiple specific files
sam run configs/agents/agent1.yaml configs/gateways/webui.yaml

# Run all files in a directory
sam run configs/agents/
```

### Skip Specific Files

Exclude files from the run:

```bash
# Skip a specific agent
sam run -s my_agent.yaml

# Skip multiple files
sam run -s agent1.yaml -s agent2.yaml
```

### Use System Environment Variables

By default, SAM loads `.env`. To use only system environment variables:

```bash
sam run -u
# or
sam run --system-env
```

## Understanding the Startup Sequence

When you run `sam run`, the following happens:

```
1. Environment Loading
   └── Load .env file (unless --system-env)
   
2. Configuration Discovery
   └── Find all .yaml/.yml files in configs/
   
3. Shared Config Loading
   └── Load shared_config.yaml
   └── Substitute environment variables
   
4. Component Initialization
   ├── Initialize broker connection
   ├── Start session services
   └── Start artifact services
   
5. Agent Startup
   ├── Load agent configurations
   ├── Initialize ADK runtime
   ├── Register tools
   └── Begin agent discovery broadcasting
   
6. Gateway Startup
   ├── Load gateway configurations
   ├── Start HTTP servers (if applicable)
   └── Begin listening for requests
   
7. Ready
   └── Application accepting requests
```

## Monitoring Runtime Behavior

### Console Output

SAM provides structured logging to the console:

```
INFO - Starting Solace Agent Mesh...
INFO - Loading configuration from configs/
INFO - Connecting to broker at tcp://localhost:55555
INFO - Agent 'main-orchestrator' initialized
INFO - Agent 'my-agent' initialized
INFO - Gateway 'webui-gateway' started on http://0.0.0.0:8000
INFO - Ready to accept requests
```

### Log Levels

Control verbosity with the `LOG_LEVEL` environment variable:

```bash
# .env
LOG_LEVEL=DEBUG   # Most verbose - shows all details
LOG_LEVEL=INFO    # Normal operation - startup, requests
LOG_LEVEL=WARNING # Only warnings and errors
LOG_LEVEL=ERROR   # Only errors
```

Or set at runtime:

```bash
LOG_LEVEL=DEBUG sam run
```

### Debug Mode

For maximum visibility during development:

```bash
# Enable debug logging
LOG_LEVEL=DEBUG sam run

# Or in .env
LOG_LEVEL=DEBUG
```

Debug mode shows:
- All message passing between agents
- Tool invocations and results
- LLM API requests/responses
- Broker message details

## Accessing the Web UI

If you configured a Web UI gateway:

1. Start the application: `sam run`
2. Open browser to `http://localhost:8000` (or your configured port)
3. The chat interface allows interaction with your agent mesh

Check the port in your `.env`:
```bash
FASTAPI_PORT=8000  # Default Web UI port
```

## Serving Documentation Locally

View SAM documentation offline:

```bash
sam docs
```

Opens documentation at `http://localhost:8585`. Change port with:

```bash
sam docs -p 9000
```

## Common Runtime Issues and Solutions

### Issue: "sam: command not found"

**Cause**: Virtual environment not activated or SAM not installed.

**Solution**:
```bash
# Activate virtual environment
source .venv/bin/activate

# Verify installation
pip show solace-agent-mesh
sam --version
```

### Issue: Broker Connection Failed

**Symptoms**:
```
ERROR - Failed to connect to broker: Connection refused
```

**Solutions**:

1. **Check broker is running**:
   ```bash
   # For Docker broker
   docker ps | grep solace
   
   # Start if not running
   docker start solace-pubsub
   ```

2. **Verify broker URL**:
   ```bash
   # In .env
   SOLACE_BROKER_URL=tcp://localhost:55555
   ```

3. **Test connectivity**:
   ```bash
   telnet localhost 55555
   # or
   nc -zv localhost 55555
   ```

4. **Check credentials**:
   ```bash
   # In .env
   SOLACE_BROKER_USERNAME=admin
   SOLACE_BROKER_PASSWORD=admin
   ```

### Issue: LLM API Errors

**Symptoms**:
```
ERROR - LLM request failed: 401 Unauthorized
ERROR - LLM request failed: Rate limit exceeded
```

**Solutions**:

1. **Verify API key**:
   ```bash
   # Test API key directly
   curl -H "Authorization: Bearer $LLM_SERVICE_API_KEY" \
        "$LLM_SERVICE_ENDPOINT/models"
   ```

2. **Check model names**:
   ```bash
   # Ensure model names match provider exactly
   LLM_SERVICE_PLANNING_MODEL_NAME=gpt-4o  # Not "GPT-4o" or "gpt4o"
   ```

3. **Rate limiting**: Add delays or reduce concurrent requests

### Issue: Port Already in Use

**Symptoms**:
```
ERROR - Address already in use: 0.0.0.0:8000
```

**Solutions**:

1. **Find process using port**:
   ```bash
   lsof -i :8000
   # or
   netstat -tlnp | grep 8000
   ```

2. **Kill the process**:
   ```bash
   kill -9 <PID>
   ```

3. **Or change the port**:
   ```bash
   # In .env
   FASTAPI_PORT=8001
   ```

### Issue: Agent Not Responding

**Symptoms**: Messages sent but no response from agent.

**Solutions**:

1. **Check agent logs**:
   ```bash
   LOG_LEVEL=DEBUG sam run
   # Look for agent activity
   ```

2. **Verify agent discovery**:
   ```yaml
   # In agent config
   discovery:
     enabled: true
   ```

3. **Check agent card publishing**:
   - Agents must publish their cards for orchestrator to find them

4. **Test direct agent communication**:
   - Use debug tools to send test messages

### Issue: Configuration Not Loading

**Symptoms**: Changes to config files not reflected.

**Solutions**:

1. **Restart the application** (configs loaded at startup):
   ```bash
   # Stop with Ctrl+C, then restart
   sam run
   ```

2. **Validate YAML syntax**:
   ```bash
   python -c "import yaml; yaml.safe_load(open('configs/agents/my_agent.yaml'))"
   ```

3. **Check file paths**:
   ```yaml
   # Relative paths are from the config file location
   shared_config_path: ../shared_config.yaml
   ```

### Issue: Tool Execution Failures

**Symptoms**:
```
ERROR - Tool execution failed: ModuleNotFoundError
```

**Solutions**:

1. **Check tool is installed**:
   ```bash
   pip list | grep <package-name>
   ```

2. **Verify tool path in config**:
   ```yaml
   tools:
     custom:
       - path: custom_components/my_tools.py
         class: MyTool
   ```

3. **Check tool implementation**:
   ```bash
   python -c "from custom_components.my_tools import MyTool"
   ```

## Debugging Techniques

### 1. Enable Verbose Logging

```bash
LOG_LEVEL=DEBUG sam run 2>&1 | tee debug.log
```

### 2. Trace Agent Communication

Look for A2A protocol messages in debug logs:
```
DEBUG - Sending A2A message to agent: my-agent
DEBUG - Received A2A response from: my-agent
```

### 3. Monitor Tool Invocations

```
DEBUG - Tool invocation: search_database
DEBUG - Tool parameters: {"query": "SELECT * FROM users"}
DEBUG - Tool result: [{"id": 1, "name": "Alice"}]
```

### 4. Check LLM Interactions

```
DEBUG - LLM request: model=gpt-4o, tokens=150
DEBUG - LLM response: 200 OK, tokens_used=75
```

### 5. Inspect Broker Messages

For advanced debugging, use Solace PubSub+ Manager:
- Web UI at `http://localhost:8080` (for Docker broker)
- Monitor queues, topics, and message flow

## Performance Monitoring

### Watch Resource Usage

```bash
# Monitor while running
watch -n 1 'ps aux | grep -E "(sam|python)" | head -10'
```

### Log Response Times

Enable timing in logs:
```python
# In custom code
import time
start = time.time()
# ... operation ...
logger.info(f"Operation took {time.time() - start:.2f}s")
```

### Profile Slow Operations

```bash
# Run with profiling
python -m cProfile -o profile.stats -m solace_agent_mesh.cli run
```

## Graceful Shutdown

Stop the application cleanly:

1. **Ctrl+C** in terminal - sends SIGINT
2. SAM performs cleanup:
   - Disconnects from broker
   - Closes HTTP servers
   - Flushes logs

For scripts:
```bash
# Find PID and send signal
kill -SIGTERM $(pgrep -f "sam run")
```

## Running in Background

### Using nohup

```bash
nohup sam run > sam.log 2>&1 &
echo $! > sam.pid
```

### Using screen/tmux

```bash
# Start in screen session
screen -S sam
sam run
# Detach: Ctrl+A, then D
# Reattach: screen -r sam
```

### Using systemd (Production)

Create `/etc/systemd/system/sam.service`:
```ini
[Unit]
Description=Solace Agent Mesh
After=network.target

[Service]
Type=simple
User=sam
WorkingDirectory=/opt/sam
ExecStart=/opt/sam/.venv/bin/sam run
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable sam
sudo systemctl start sam
sudo systemctl status sam
```

## Common Patterns

### Development Workflow

```bash
# 1. Make changes to config/code
# 2. Stop running instance (Ctrl+C)
# 3. Restart with debug logging
LOG_LEVEL=DEBUG sam run

# 4. Test changes
# 5. Repeat
```

### Testing Individual Agents

```bash
# Run only the agent you're testing
sam run configs/agents/my_agent.yaml configs/shared_config.yaml
```

### Hot Reload (Development)

SAM doesn't support hot reload natively. Use a file watcher:

```bash
# Install watchdog
pip install watchdog

# Watch for changes and restart
watchmedo auto-restart --pattern="*.yaml;*.py" --recursive -- sam run
```

## Related Skills

- [initialize-project](initialize-project-skill.md) - Project setup
- [configure-shared-settings](configure-shared-settings-skill.md) - Configuration management
- [create-and-manage-agents](create-and-manage-agents-skill.md) - Agent development
