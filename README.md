# MCP Server Setup with Python & Kiro CLI

A guide to setting up Model Context Protocol (MCP) servers with Python virtual environments and configuring them with Kiro CLI.

## Prerequisites

- Python 3.8+
- Kiro CLI installed
- `uv` (fast Python package installer)

## Project Setup

### 1. Initialize Project & Create Virtual Environment

```bash
# Initialize project
uv init weather

# Create virtual environment
uv venv

# Activate environment
source .venv/bin/activate
```

### 2. Install Dependencies

```bash
uv add "httpx>=0.28.1"
uv add "mcp[cli]>=1.25.0"
```

### 3. Create MCP Server

Create your MCP server (`weather.py`):

```python
from mcp import Server
import asyncio

server = Server("weather-mcp-server")

@server.tool()
def get_forecast(location: str) -> str:
    """Get weather forecast for a location"""
    return f"Forecast for {location}: Sunny, 72°F"

@server.tool()
def get_alerts(location: str) -> str:
    """Get weather alerts for a location"""
    return f"No alerts for {location}"

async def main():
    await server.run()

if __name__ == "__main__":
    asyncio.run(main())
```

### 4. Configure Kiro CLI Agent

Create an agent configuration file `agents/weather.json`:

```json
{
  "name": "weather",
  "description": "Weather agent with access to weather forecast and alerts",
  "tools": [
    "*"
  ],
  "mcpServers": {
    "weather": {
      "command": "uv",
      "args": ["run", "weather.py"],
      "cwd": "/path/to/your/weather/project"
    }
  },
  "allowedTools": ["@weather/get_forecast", "@weather/get_alerts"]
}
```

**Important**: Update the `cwd` path to match your actual project location.

### 5. Use the Agent

```bash
# Start Kiro CLI with the weather agent
kiro-cli chat --agent agents/weather.json

# Your MCP tools are now available in the chat
```

## Dependencies Used

- `httpx>=0.28.1` - HTTP client for async requests
- `mcp[cli]>=1.25.0` - Model Context Protocol with CLI tools

## Agent Configuration Explained

- `name` - Agent identifier
- `description` - What the agent does
- `tools` - Available tools (`"*"` means all tools)
- `mcpServers` - MCP server configuration
  - `command` - Command to run (`uv` for this project)
  - `args` - Arguments to pass (`["run", "weather.py"]`)
  - `cwd` - Working directory (absolute path to your project)
- `allowedTools` - Specific tools the agent can use

## Troubleshooting

- Ensure virtual environment is activated with `source .venv/bin/activate`
- Check dependencies are installed correctly with `uv pip list`
- Verify the `cwd` path in `weather.json` is correct
- Make sure `weather.py` is executable
