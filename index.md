---
title: MCP over MQTT
---

# MCP over MQTT

This is the specification and SDK implementations of MCP over MQTT, an transport layer for the [Model Context Protocol](https://modelcontextprotocol.io).

## Why MQTT?

MQTT is a lightweight and widely used protocol for IoT and edge computing. It is designed for unreliable networks and low bandwidth, making it a good choice for edge devices and cloud services to communicate with each other.

Introducing MQTT as a transport layer for MCP enables the protocol to be used in a wider range of applications, including edge computing, IoT, and cloud services — anywhere MQTT is to be used.

## Features

The MQTT transport protocol supports all the features of MCP and adds the following features:

- **Built-in Service Registry and Discovery**: MCP clients can discover available MCP servers from the MQTT broker.

- **Built-in Load Balancing and Scalability**: MCP servers can be scaled horizontally by adding more MCP server instances, while keeping the the MCP server side stateful.

Additionally, by setting access control permissions on the MQTT topics used by MCP clients and servers in the MQTT broker, authorization for clients and servers can be very flexibly implemented.

## Limitations

MCP over MQTT is designed for remotely deployed MCP servers, and the protocol requires a centralized MQTT broker to function. Although locally deployed MCP servers can also use the MCP over MQTT, it may introduce additional deployment complexity.

We are not meant to replace the existing MCP transport layers (stdio and HTTP SSE), but to provide an alternative for edge computing and IoT applications, and for the backend services that want to use MQTT as their messaging protocol.