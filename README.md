# M365 Copilot Proxy

A lightweight bridge that allows OpenAI-compatible coding agents to use
Microsoft 365 Copilot as an LLM backend.

The proxy translates Microsoft 365 Copilot's WebSocket/SignalR
communication layer into an OpenAI-compatible API, enabling tools such
as `pi` and `OpenClaw` to interact with Copilot through familiar
endpoints.

## Overview

Microsoft 365 Copilot does not expose a native OpenAI API. This project
provides a compatibility layer with:

-   OpenAI-compatible chat completion endpoints
-   Model discovery support
-   Tool calling translation
-   Copilot Studio agent integration
-   Session reuse for efficient conversations
-   Image generation support
-   Multiple client integrations

## Architecture

The system consists of three main components:

### 1. Standalone Proxy

Provides HTTP endpoints compatible with OpenAI clients:

-   `/v1/chat/completions`
-   `/v1/models`

Any compatible application can connect using the proxy as its API
endpoint.

### 2. OpenClaw Integration

Includes:

-   Provider configuration generation
-   Setup utilities
-   OpenClaw-specific integration support

### 3. Core Package

Shared functionality including:

-   Authentication handling
-   WebSocket communication
-   Tool formatting
-   Agent management
-   Conversation handling

## Tool Calling

Microsoft 365 Copilot does not natively use OpenAI-style `tool_calls`.

This project implements a translation layer:

1.  Tool definitions are injected into prompts.
2.  Copilot generates structured markdown-fenced tool requests.
3.  The proxy parses these requests.
4.  Requests are converted into OpenAI-compatible tool calls.

The proxy also supports shell-based routing, allowing coding agents to
execute multi-step tasks by generating shell commands through supported
tools.

## Agent Mode

For improved reliability, the proxy can create and use a Copilot Studio
agent.

The setup process:

1.  Detects the Power Platform environment.
2.  Creates a Copilot Studio bot.
3.  Publishes the agent.
4.  Uses the generated agent identifier for WebSocket requests.
5.  Stores the agent ID locally for future sessions.

## Conversation Handling

Sessions reuse the same Microsoft 365 conversation context.

Benefits:

-   Lower token/message usage
-   Persistent multi-turn interactions
-   Reduced backend overhead

## Installation

### Requirements

-   Node.js 24+
-   pnpm 10+
-   Microsoft 365 account with Copilot access

### Install

``` bash
git clone https://github.com/cramt/m365-copilot-proxy
cd m365-copilot-proxy

pnpm install
pnpm build
```

## Configuration

Create:

    ~/.config/opencode-m365/secrets.json

Example:

``` json
{
  "email": "you@company.com",
  "password": "your-password",
  "mfaSecret": "YOUR_TOTP_SECRET"
}
```

The `mfaSecret` value must be the Base32 TOTP seed, not a generated
authentication code.

## Running the Proxy

Start the proxy:

``` bash
m365-proxy 4141
```

The OpenAI-compatible endpoint will be available at:

    http://localhost:4141/v1

## Using with Coding Agents

### Example: pi

Configure the provider:

``` json
{
  "providers": {
    "m365": {
      "baseUrl": "http://localhost:4141/v1",
      "api": "openai-completions",
      "apiKey": "not-needed"
    }
  }
}
```

Run:

``` bash
pi --models "gpt-5.5-think-deeper"
```

### Example: OpenClaw

Setup:

``` bash
m365-openclaw-setup --start
```

## Supported Models

Examples of supported model routes:

  Model                    Purpose
  ------------------------ -----------------------------------------
  `gpt-5.5-think-deeper`   Recommended for coding agents and tools
  `gpt-5.6-think-deeper`   Advanced reasoning
  `gpt-5.6`                General chat
  `gpt-6-think-deeper`     Premium reasoning model
  Claude-based models      Supported through Copilot routing

Model availability depends on Microsoft 365 account entitlements.

## Image Generation

The proxy can expose Copilot image generation capabilities.

Example:

``` ts
generateImage(
  "A minimalist flat-design logo of a lighthouse"
)
```

Generated images are returned through the supported API layer.

## Authentication Flow

Authentication supports:

1.  Cached MSAL token refresh
2.  Automated browser login
3.  Interactive approval fallback

Authentication data is stored locally under:

    ~/.config/opencode-m365/

## Environment Variables

Common options:

  ------------------------------------------------------------------------
  Variable                             Description
  ------------------------------------ -----------------------------------
  `M365_DEBUG`                         Enable debug logging

  `M365_TRACE`                         Enable detailed protocol tracing

  `M365_ENABLE_INTERACTIVE_APPROVAL`   Allow manual browser authentication

  `M365_FORCE_AGENT`                   Control Copilot Studio agent usage

  `M365_CACHE_FILE`                    Override token cache location
  ------------------------------------------------------------------------

## Development

``` bash
pnpm install
pnpm build
pnpm run dev
pnpm run test:unit
pnpm run test:live
```

## Limitations

Current limitations:

-   Tool calling is implemented through translation rather than native
    function calling.
-   Large tool payloads may trigger Copilot disengagement behavior.
-   Some advanced models require premium Copilot access.
-   Conversation limits are controlled by Microsoft 365 backend
    policies.
-   Tool responses may require buffering before delivery.

## License

MIT License.

Use responsibly. This project communicates with Microsoft services using
your own account and credentials, subject to your organization's
Microsoft 365 policies.
