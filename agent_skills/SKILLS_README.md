# Solace Agent Mesh - Agent Skills

This directory contains **Agent Skills** for AI coding assistants to effectively work with Solace Agent Mesh. These skills follow the [AgentSkills.io specification](https://agentskills.io/specification) and are designed to be consumed by tools like Claude Code, Cursor, Gemini CLI, VS Code Copilot, and other agentic coding environments.

## What are Agent Skills?

Agent Skills are structured markdown documents that teach AI coding assistants how to perform specific tasks. They provide:
- Step-by-step procedural knowledge
- Code examples and configuration templates
- Troubleshooting guidance
- Best practices and common patterns

## Available Skills

| Skill | Description | Complexity |
|-------|-------------|------------|
| [initialize-project](initialize-project-skill.md) | Set up new SAM projects with broker, LLM, and component configuration | Beginner |
| [configure-shared-settings](configure-shared-settings-skill.md) | Multi-environment YAML configuration and environment variables | Intermediate |
| [run-and-debug-projects](run-and-debug-projects-skill.md) | Execute, monitor, and troubleshoot SAM applications | Beginner |
| [create-and-manage-agents](create-and-manage-agents-skill.md) | Agent development, configuration, and A2A communication | Intermediate |
| [configure-agent-tools](configure-agent-tools-skill.md) | Built-in, custom Python, and MCP tool integration | Intermediate |
| [create-and-manage-gateways](create-and-manage-gateways-skill.md) | Gateway creation for REST, SSE, webhooks, and custom interfaces | Intermediate |
| [manage-plugins](manage-plugins-skill.md) | Plugin discovery, installation, creation, and publishing | Intermediate |

## Quick Start

For new users, follow the skills in this order:
1. **initialize-project** - Get your first SAM project running
2. **run-and-debug-projects** - Learn to run and monitor your application
3. **create-and-manage-agents** - Add custom agents to your mesh
4. **configure-agent-tools** - Equip agents with capabilities
5. **create-and-manage-gateways** - Expose your mesh to external systems

## Related Resources

- **Context7**: For quick command reference, see [context7.com/solacelabs/solace-agent-mesh](https://context7.com/solacelabs/solace-agent-mesh)
- **Documentation**: Full docs at [docs.solace.com](https://docs.solace.com/solace-agent-mesh/)
- **GitHub**: [github.com/SolaceLabs/solace-agent-mesh](https://github.com/SolaceLabs/solace-agent-mesh)

## For AI Coding Assistants

When working with Solace Agent Mesh:
1. **Always use a Python virtual environment** (`python -m venv .venv`)
2. **Install with pip**: `pip install solace-agent-mesh`
3. **Use the `sam` CLI** for all project management tasks
4. **Refer to templates in `templates/` and examples in `examples/`** for configuration patterns

## License

These skills are part of the Solace Agent Mesh project and are licensed under the Apache 2.0 License.
