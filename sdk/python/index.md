---
title: Python SDK
---

# Quickstart

## Create a Demo Project

Now let create a demo project using [uv](https://docs.astral.sh/uv/):

```bash
uv init mcp_over_mqtt_demo
cd mcp_over_mqtt_demo
```

## Create a Simple MCP Server

In the `mcp_over_mqtt_demo` project, let's create a simple MCP server that exposes a calculator tool and some resources. Create a file named `demo_mcp_server.py` in the project directory and add the following code:

```python
# demo_mcp_server.py
from mcp.server.fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP(
    "demo_mcp_server/calculator",
    log_level="DEBUG",
    mqtt_server_description="A simple FastMCP server that exposes a calculator tool",
    mqtt_options={
        "host": "broker.emqx.io",
    },
)

# Add an addition tool
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

# Add a dynamic greeting resource
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Get a personalized greeting"""
    return f"Hello, {name}!"
```

## Create a Simple MCP Client

In the same project, let's create a simple MCP client that connects to the server and lists the available tools and resources. Create a file named `demo_mcp_client.py` in the project directory and add the following code:

```python
# demo_mcp_client.py
import logging
import anyio
import mcp.client.mqtt as mcp_mqtt
from mcp.shared.mqtt import configure_logging

configure_logging(level="INFO")
logger = logging.getLogger(__name__)

async def on_mcp_server_discovered(client, server_name):
    logger.info(f"Discovered {server_name}, connecting ...")
    await client.initialize_mcp_server(server_name)

async def on_mcp_connect(client, server_name, connect_result):
    success, init_result = connect_result
    if success == 'error':
        logger.error(f"Failed to connect to {server_name}: {init_result}")
        return
    logger.info(f"Connected to {server_name}, success={success}, init_result={init_result}")
    capabilities = init_result.capabilities
    if capabilities.prompts:
        prompts = await client.list_prompts(server_name)
        logger.info(f"Prompts of {server_name}: {prompts}")
    if capabilities.resources:
        resources = await client.list_resources(server_name)
        logger.info(f"Resources of {server_name}: {resources}")
        resource_templates = await client.list_resource_templates(server_name)
        logger.info(f"Resources templates of {server_name}: {resource_templates}")
    if capabilities.tools:
        toolsResult = await client.list_tools(server_name)
        tools = toolsResult.tools
        logger.info(f"Tools of {server_name}: {tools}")
        if tools[0].name == "add":
            result = await client.call_tool(server_name, name = tools[0].name, arguments={"a": 1, "b": 2})
            logger.info(f"Calling the tool as add(a=1, b=2), result: {result}")

async def on_mcp_disconnect(client, server_name):
    logger.info(f"Disconnected from {server_name}")

async def main():
    async with mcp_mqtt.MqttTransportClient(
        "test_client",
        auto_connect_to_mcp_server = True,
        on_mcp_server_discovered = on_mcp_server_discovered,
        on_mcp_connect = on_mcp_connect,
        on_mcp_disconnect = on_mcp_disconnect,
        mqtt_options = mcp_mqtt.MqttOptions(
            host="broker.emqx.io",
        )
    ) as client:
        client.start()
        while True:
            ## Simulate other works while the MQTT transport client is running in the background...
            await anyio.sleep(20)

if __name__ == "__main__":
    anyio.run(main)
```

## Run the Demo

1. First, install the required dependencies:

```bash
uv add git+https://github.com/emqx/mcp-python-sdk --branch main
uv add "mcp[cli]"
```

2. Now run the client:

```bash
uv run demo_mcp_client.py
```

3. Open a new terminal and run the server:

```bash
uv run mcp run --transport mqtt ./demo_mcp_server.py
```

Although the client was started first, it will discover the servers and connect to it. The client will then list the available tools and resources, and call the `add` tool with the arguments `a=1` and `b=2`.
