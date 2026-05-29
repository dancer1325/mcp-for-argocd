# Argo CD MCP Server

* == [MCP](https://modelcontextprotocol.io) server implementation -- for -- [Argo CD](https://argo-cd.readthedocs.io/en/stable/)
  * allow
    * AI assistants can interact -- ,through natural language,with -- your Argo CD applications 
  * provide
    * seamless integration -- with --
      * Visual Studio Code
      * other MCP clients -- through --
        * stdio
        * HTTP stream

[<img src="https://img.shields.io/badge/VS_Code-VS_Code?style=flat-square&label=Install%20Server&color=0098FF" alt="Install in VS Code">](https://insiders.vscode.dev/redirect?url=vscode%3Amcp%2Finstall%3F%257B%2522name%2522%253A%2522argocd-mcp%2522%252C%2522command%2522%253A%2522npx%2522%252C%2522args%2522%253A%255B%2522argocd-mcp%2540latest%2522%252C%2522stdio%2522%255D%252C%2522env%2522%253A%257B%2522ARGOCD_BASE_URL%2522%253A%2522%253Cargocd_url%253E%2522%252C%2522ARGOCD_API_TOKEN%2522%253A%2522%253Cargocd_token%253E%2522%257D%257D)  [<img alt="Install in VS Code Insiders" src="https://img.shields.io/badge/VS_Code_Insiders-VS_Code_Insiders?style=flat-square&label=Install%20Server&color=24bfa5">](https://insiders.vscode.dev/redirect?url=vscode-insiders%3Amcp%2Finstall%3F%257B%2522name%2522%253A%2522argocd-mcp%2522%252C%2522command%2522%253A%2522npx%2522%252C%2522args%2522%253A%255B%2522argocd-mcp%2540latest%2522%252C%2522stdio%2522%255D%252C%2522env%2522%253A%257B%2522ARGOCD_BASE_URL%2522%253A%2522%253Cargocd_url%253E%2522%252C%2522ARGOCD_API_TOKEN%2522%253A%2522%253Cargocd_token%253E%2522%257D%257D)

![argocd-mcp-demo](https://github.com/user-attachments/assets/091548d0-9927-4d4b-a2fe-4f99c7cea108)

## Features

- **Transport Protocols**
  - supported modes
    - stdio
    - HTTP stream 
  - enable
    - flexible integration -- with -- DIFFERENT clients
- **Complete Argo CD API Integration**
- **AI Assistant Ready**
  - == Pre-configured AI assistant's tools -- , to interact, vía natural language, -- with Argo CD

## Available Tools

* by default, 
  * ALL are AVAILABLE

### Cluster Management
- `list_clusters`: List all clusters registered with ArgoCD

### Application Management
- `list_applications`: List and filter all applications
- `get_application`: Get detailed information about a specific application
- `create_application`: Create a new application
- `update_application`: Update an existing application
- `delete_application`: Delete an application
- `sync_application`: Trigger a sync operation on an application

### Resource Management
- `get_application_resource_tree`: Get the resource tree for a specific application
- `get_application_managed_resources`: Get managed resources for a specific application
- `get_application_workload_logs`: Get logs for application workloads (Pods, Deployments, etc.)
- `get_resource_events`: Get events for resources managed by an application
- `get_resource_actions`: Get available actions for resources
- `run_resource_action`: Run an action on a resource

## Installation

### Prerequisites

- Node.js v18+
- pnpm package manage 
  - uses
    - development
- Argo CD instance / enable API access
- [Argo CD API token](https://argo-cd.readthedocs.io/en/stable/developer-guide/api-docs/#authorization)

### Usage with Cursor
1. [Cursor documentation for MCP support](https://docs.cursor.com/context/model-context-protocol)
2. create ".cursor/mcp.json"
```json
{
  "mcpServers": {
    "argocd-mcp": {
      "command": "npx",
      "args": [
        "argocd-mcp@latest",
        "stdio"
      ],
      "env": {
        "ARGOCD_BASE_URL": "<argocd_url>",
        "ARGOCD_API_TOKEN": "<argocd_token>"
      }
    }
  }
}
```

3. Start a conversation with Agent mode

### Usage with VSCode

1. [how to use MCP servers | VS Code documentation](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
2. create ".vscode/mcp.json"
```json
{
  "servers": {
    "argocd-mcp-stdio": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "argocd-mcp@latest",
        "stdio"
      ],
      "env": {
        "ARGOCD_BASE_URL": "<argocd_url>",
        "ARGOCD_API_TOKEN": "<argocd_token>"
      }
    }
  }
}
```

3. start a conversation / AI assistant | VS Code

### Usage with Claude Desktop

1. [MCP | Claude Desktop](https://modelcontextprotocol.io/quickstart/user)
2. create a "claude_desktop_config.json"
```json
{
  "mcpServers": {
    "argocd-mcp": {
      "command": "npx",
      "args": [
        "argocd-mcp@latest",
        "stdio"
      ],
      "env": {
        "ARGOCD_BASE_URL": "<argocd_url>",
        "ARGOCD_API_TOKEN": "<argocd_token>"
      }
    }
  }
}
```

3. Configure Claude Desktop / use this configuration file | settings

### Self-signed Certificates

If your Argo CD instance uses self-signed certificates or certificates from a private Certificate Authority (CA), you may need to add the following environment variable to your configuration:

```
"NODE_TLS_REJECT_UNAUTHORIZED": "0"
```

This disables TLS certificate validation for Node.js when connecting to Argo CD instances using self-signed certificates or certificates from private CAs that aren't trusted by your system's certificate store.

> **Warning**: Disabling SSL verification reduces security. Use this setting only in development environments or when you understand the security implications.


### Read Only Mode

- steps to configure It
  - set the "MCP_READ_ONLY": "true" environment variable

- disable
  - `create_application`
  - `update_application`
  - `delete_application`
  - `sync_application`
  - `run_resource_action`

### Stateless Mode

By default, the HTTP transport assigns a session ID to each client connection and keeps an in-memory map of active sessions. This works well for single-instance deployments but causes `400` errors when multiple replicas are running without sticky sessions, because a request routed to a different pod will not find the session that was created on the original pod.

- if you want to run WITHOUT session affinity requirements -> start the server with the `--stateless` flag
  - | node

```bash
node dist/index.js http --stateless
```

  - | Docker

```bash
docker run -e ARGOCD_BASE_URL=<argocd_url> -e ARGOCD_API_TOKEN=<argocd_token> \
  argoprojlabs/mcp-for-argocd http --stateless
```

- NO return NOR require `Mcp-Session-Id`
  - -> replica can handle ANY request
- requirements
  - supply ArgoCD credentials / EACH request
    - ways to provide
      - -- via -- environment variables, OR
      - `x-argocd-base-url` / `x-argocd-api-token` headers
- `GET /mcp` & `DELETE /mcp`
  - 's return: `405 Method Not Allowed`
    - Reason: 🧠 NOT support
      - session-level SSE 
      - termination 🧠

- use cases
  - Kubernetes deployments / 
    - have Horizontal Pod Autoscaling (HPA)  
    - network-level sticky sessions are NOT AVAILABLE 

## how to develop?

- `pnpm install`
- `pnpm run dev`
- use your MCP server

### Upgrading ArgoCD Types

To update the TypeScript type definitions based on the latest Argo CD API specification:

1. Download the `swagger.json` file from the [ArgoCD release page](https://github.com/argoproj/argo-cd/releases), for example here is the [swagger.json link](https://github.com/argoproj/argo-cd/blob/v2.14.11/assets/swagger.json) for ArgoCD v2.14.11.

2. Place the downloaded `swagger.json` file in the root directory of the `argocd-mcp` project.

3. Generate the TypeScript types from the Swagger definition by running the following command. This will create or overwrite the `src/types/argocd.d.ts` file:
    ```bash
    pnpm run generate-types
    ```

4. Update the `src/types/argocd-types.ts` file to export the required types from the newly generated `src/types/argocd.d.ts`. This step often requires manual review to ensure only necessary types are exposed.

