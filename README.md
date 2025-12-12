# Exposing Existing APIs as MCP Servers

This repository provides comprehensive documentation on exposing existing APIs as Model Context Protocol (MCP) servers across different platforms and tools. MCP servers enable AI assistants to securely interact with your APIs through a standardized protocol.

## Quick Start Guide

Choose your platform based on your existing infrastructure:

| Platform | Prerequisites | Setup Time |
|----------|---------------|------------|
| [Postman](#postman) | Node.js 18+ | 10 minutes |
| [Google Cloud (Apigee)](#google-cloud-platform-apigee) | GCP Project, Apigee X | 30 minutes |
| [Microsoft Azure](#microsoft-azure) | Azure APIM instance | 20 minutes |
| [AWS](#aws) | Python 3.10+, AWS CLI | 15 minutes |

## Glossary

- **MCP (Model Context Protocol)**: A standardized protocol for AI assistants to interact with external tools and APIs
- **MCP Server**: A service that exposes APIs as tools that AI assistants can discover and invoke
- **MCP Client**: An application (like Claude Desktop or GitHub Copilot) that connects to MCP servers
- **stdio**: Standard input/output communication method for local MCP servers
- **SSE (Server-Sent Events)**: HTTP-based streaming protocol for remote MCP servers
- **SigV4**: AWS Signature Version 4, an authentication protocol for AWS services
- **Tool**: An API operation exposed through MCP that AI assistants can invoke

## Table of Contents

- [Postman](#postman)
- [Google Cloud Platform (Apigee)](#google-cloud-platform-apigee)
- [Microsoft Azure](#microsoft-azure)
- [AWS](#aws)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)

---

## <img src="https://avatars.githubusercontent.com/u/10251060?s=24" width="24" height="24" alt="Postman" /> Postman

Postman provides an MCP Generator that allows you to create MCP servers from public APIs available in the Postman API Network.

### Prerequisites

- Node.js 18 or later installed
- Postman account with access to API Network
- Basic understanding of REST APIs

### Generate an MCP Server

1. From the Postman header, click **API Network**
2. From the left sidebar, click **MCP Generator**
3. Search for a public API from the Postman API Network
4. Choose a public workspace from the search results
5. Navigate through the workspace's collections (groups of related API requests) and folders
6. Select the specific API requests you want to add to your MCP server
7. Click **Add Requests**
8. Optionally, search and add requests from other public workspaces to combine multiple APIs
9. Click **Generate** to create the MCP server package
10. Click **Download ZIP** and follow the onscreen instructions

### Set Up Your MCP Server

1. Unzip the downloaded file to your desired location
2. Open terminal and navigate to the MCP server's root directory:
```bash
   cd /path/to/your-mcp-server
```
3. Install dependencies:
```bash
   npm install
```
4. List available tools to verify setup:
```bash
   npm run list-tools
```
   This command lists your tool's information, including their file names. You can find these files in the `tools/` directory.
5. If your APIs require authentication, store sensitive data such as API keys or tokens in the `.env` file:
```env
   API_KEY=your_api_key_here
   API_SECRET=your_api_secret_here
```
   Refer to your project's `README.md` file for specific configuration requirements.

### Start Your MCP Server

To start with standard input/output (for local use with Claude Desktop):
```bash
node mcpServer.js
```

To start with streamable HTTP (for remote access):
```bash
node mcpServer.js --streamable-http
```

Stop the server with `Control+C` (Mac) or `Ctrl+C` (Windows/Linux).

### Configure MCP Client

To use your Postman MCP server with Claude Desktop, you need to add it to Claude's configuration file.

**Configuration File Locations:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

**Steps to Configure:**

1. Locate and open the `claude_desktop_config.json` file in a text editor
2. Add your Postman MCP server configuration:

```json
{
  "mcpServers": {
    "postman-api": {
      "command": "node",
      "args": ["/absolute/path/to/your/mcpServer.js"]
    }
  }
}
```

3. Replace `/absolute/path/to/your/mcpServer.js` with the full path to your MCP server file
   - Example (macOS/Linux): `"/Users/username/postman-mcp-server/mcpServer.js"`
   - Example (Windows): `"C:\\Users\\username\\postman-mcp-server\\mcpServer.js"`

4. Save the configuration file
5. Restart Claude Desktop completely (quit and reopen the application)
6. Open a new conversation in Claude Desktop
7. Your Postman API tools should now be available for Claude to use

**Note:** The MCP server runs automatically when Claude Desktop starts. You don't need to manually start it with `node mcpServer.js` when using it with Claude Desktop.

### Troubleshooting

**Issue: "Cannot find module" error**
- Solution: Ensure you ran `npm install` in the MCP server directory

**Issue: API requests failing with 401 Unauthorized**
- Solution: Verify API keys are correctly set in `.env` file and authentication headers are properly configured in tool files

**Issue: Tools not appearing in Claude Desktop**
- Solution: Restart Claude Desktop after modifying configuration file, and check the path to `mcpServer.js` is absolute

---

## <img src="https://www.gstatic.com/images/branding/product/2x/google_cloud_48dp.png" width="24" height="24" alt="Google Cloud" /> Google Cloud Platform (Apigee)

Apigee now offers **native MCP support** that allows enterprises to turn their existing APIs into secure, governed MCP tools without writing code or deploying separate MCP servers.

> **Native MCP Support (Preview)**
> 
> Apigee's native MCP support is currently in **preview**. With this feature, you don't need to make any changes to your existing APIs, write any code, or deploy and manage any local or remote MCP servers. Apigee uses your existing API specifications and manages the underlying infrastructure and transcoding.
> 
> **To access this feature, contact your Apigee or Google Cloud account team.**

### Key Features

- **No code changes required**: Turn existing APIs into MCP tools using your OpenAPI specifications
- **Fully managed infrastructure**: Apigee handles MCP servers, transcoding, and protocol handling
- **Enterprise-grade security**: Extends Apigee's 30+ built-in policies for authorization, authentication, and governance to AI agents
- **Automatic API hub registration**: Deployed MCP proxies are automatically registered in Apigee API hub
- **Comprehensive observability**: Use Apigee Analytics and API Insights to monitor MCP tool usage
- **Framework compatibility**: Works with ADK, LangGraph, and other popular agent frameworks

### Key Benefits

1. **No Added Operational Burden**: You don't need to set up and manage an MCP server for each of your APIs. Just deploy an MCP proxy, and Apigee takes care of the rest—fully managing the MCP servers, transcoding, and protocol handling.

2. **Tool Observability and Governance**: Apigee's built-in identity, authorization, and security policies can secure and govern your MCP endpoints and tools. Use Apigee Analytics to monitor tool usage by MCP clients.

3. **Comprehensive Tool Security**: Apigee ensures all agentic interactions are secure:
   - Use [Cloud Data Loss Prevention (DLP)](https://cloud.google.com/security/products/dlp) to classify and protect sensitive data
   - Use [Model Armor](https://cloud.google.com/security-command-center/docs/model-armor-overview) to guard against prompt injection and jailbreaking attempts
   - Enforce proper IAM permissions for agents and users to invoke MCP tools
   - Use [Apigee Advanced API Security](https://cloud.google.com/apigee/docs/api-security) for additional protection

4. **Centralized Tool Catalog**: Deployed MCP proxies are automatically registered in Apigee API hub with your spec, allowing you to maintain a searchable, centralized tool catalog and promote tool reuse.

### How It Works (Native MCP Support)

1. **Create an MCP Proxy**: In your Apigee environment group, create an MCP proxy with:
   - Base path: `/mcp`
   - Target URL: `mcp.apigeex.com`
   - Include your OpenAPI specification

2. **Automatic Tool Generation**: When a `tools/list` or `tools/call` request is made to the MCP endpoint, Apigee uses the operations documented in your OpenAPI spec as the MCP tools list.

3. **Apply Policies**: Bundle the MCP proxy in an [API Product](https://cloud.google.com/apigee/docs/api-platform/publish/what-api-product) and apply granular quota, identity, and access policies to ensure only authorized MCP clients, agents, and developers can list and call those tools.

4. **Monitor Usage**: Use Apigee Analytics to monitor MCP tool usage, and use the "Insights" tab in Apigee API hub to view traffic and performance metrics for your MCP endpoints.

### Agent Development Kit (ADK) Integration

Developers using [Agent Development Kit (ADK)](https://google.github.io/adk-docs/) have a streamlined advantage when building agents within the Google ecosystem:

- **ADK Toolset**: ADK includes a [toolset for Apigee and Application Integration](https://google.github.io/adk-docs/tools/google-cloud-tools/), making it easy to connect custom agents to your MCP endpoints
- **ApigeeLLM Wrapper**: Use the [ApigeeLLM wrapper for ADK](https://google.github.io/adk-docs/agents/models/#using-apigee-gateway-for-ai-models) to expose your LLM endpoint through an Apigee proxy, integrating governance into your agentic workflows
- **Deployment Options**: Use [Vertex AI Agent Engine](https://cloud.google.com/agent-builder/agent-engine/overview) to deploy your agents and put them in action across your organization using [Gemini Enterprise](https://cloud.google.com/gemini-enterprise)

> **Note**: The ApigeeLLM wrapper is currently designed for use with Vertex AI and the Gemini API in Google AI Studio, with support for other models and interfaces planned.

---

### Alternative: Sample-Based Approach

If you need an immediately available solution or want more control over the MCP server implementation, you can use the [Apigee MCP Sample](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp) from Google Cloud Platform. This sample provides an MCP server implementation that dynamically discovers Apigee-managed API Products and exposes them as MCP tools.

#### Prerequisites (Sample-Based)

- Apigee X Organization with at least one environment
- GCP Project with the following APIs enabled:
  - Vertex AI API
  - Cloud Run API
  - Apigee API
- Apigee API hub enabled and provisioned in the same GCP project
- gcloud CLI installed and configured
- Docker installed (for building container images)
- apigeecli installed ([download releases](https://github.com/apigee/apigeecli/releases))
- jq (JSON processor) installed ([download](https://jqlang.org/download/))

#### Getting Started (Sample-Based)

**1. Clone the Repository:**
```bash
git clone https://github.com/GoogleCloudPlatform/apigee-samples.git
cd apigee-samples/apigee-mcp
```

**2. Configure Environment Variables:**

Edit the `apigee-mcp/env.sh` and set the following variables based on your Apigee and GCP Project details:

```bash
export PROJECT="<PROJECT_ID_TO_SET>"                    # Your GCP Project ID
export REGION="<REGION_TO_SET>"                         # e.g., us-central1
export APIGEE_ENV="<APIGEE_ENV_TO_SET>"                 # e.g., eval
export APIGEE_HOST="<APIGEE_HOST_TO_SET>"               # e.g., your-org-eval.apigee.net
export SA_EMAIL="<SA_EMAIL_TO_SET>"                     # e.g., apigee-runtime-sa@<PROJECT_ID>.iam.gserviceaccount.com
```

**Example Configuration:**
```bash
export PROJECT="my-gcp-project"
export REGION="us-central1"
export APIGEE_ENV="eval"
export APIGEE_HOST="my-org-eval.apigee.net"
export SA_EMAIL="apigee-runtime-sa@my-gcp-project.iam.gserviceaccount.com"
```

**3. Source the Environment File:**
```bash
source ./env.sh
```

**4. Deploy:**
```bash
./deploy-all.sh
```

The deployment script performs the following actions:
1. Builds container images for stub services and MCP server
2. Deploys services to Google Cloud Run
3. Configures Apigee artifacts (API Proxies, Products, Developer Apps)
4. Sets up Apigee API hub entries
5. Outputs the MCP server endpoint URL

For detailed setup instructions, refer to the [README in the repository](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp).

### Additional Resources

- [MCP Support for Apigee Blog Post](https://cloud.google.com/blog/products/ai-machine-learning/mcp-support-for-apigee)
- [Announcing MCP Support for Google Services](https://cloud.google.com/blog/products/ai-machine-learning/announcing-official-mcp-support-for-google-services)
- [Apigee MCP Sample Repository](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp)
- [Agent Development Kit (ADK) Documentation](https://google.github.io/adk-docs/)
- [Apigee API Hub Documentation](https://cloud.google.com/apigee/docs/apihub/getting-started-apihub)
- [Apigee Documentation](https://cloud.google.com/apigee/docs)

### Troubleshooting

**Issue: "Failed to fetch API products"**
- Solution: Verify `MCP_BASE_URL` points to a valid Apigee API hub endpoint and credentials are correct

**Issue: Container fails to start (sample-based approach)**
- Solution: Check Cloud Run logs with `gcloud run services logs read mcp-server` and verify all required environment variables are set

**Issue: OAuth authentication failing**
- Solution: Ensure the Developer App in Apigee has the correct credentials and the API Product is associated with it

**Issue: MCP tools not appearing for agents**
- Solution: Verify your OpenAPI specification is valid and operations are properly documented

**Issue: Access denied when calling MCP tools**
- Solution: Check that the API Product includes the MCP proxy and the client has proper credentials

For more troubleshooting help, see the [repository's troubleshooting section](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp#troubleshooting)

---

## <img src="https://azure.microsoft.com/svghandler/azure/?width=24&height=24" width="24" height="24" alt="Microsoft Azure" /> Microsoft Azure

Azure API Management allows you to expose REST APIs as remote MCP servers using its built-in AI gateway capabilities.

### Prerequisites

- Azure API Management instance that supports AI Gateway.
  - **Recommended Tiers**: `Basic v2`, `Standard v2`, or `Premium v2` natively include the AI Gateway feature.
  - **Classic Tiers**: `Basic`, `Standard`, and `Premium` require joining the AI Gateway Early Access program.
  - > If you need to create a new instance, follow the [official Azure documentation](https://docs.microsoft.com/azure/api-management/get-started-create-service-instance) and choose a `v2` tier for immediate access to AI Gateway.
- HTTP-compatible REST API managed in API Management
- Visual Studio Code with GitHub Copilot extension (for testing)
- Azure subscription with appropriate permissions

### Joining AI Gateway Early Access

For classic tiers (Basic, Standard, Premium), you must join the AI Gateway Early Access group:

1. Navigate to your API Management instance in Azure Portal
2. Go to **Settings** > **Features**
3. Find **AI Gateway** and click **Join Early Access**
4. Wait up to 2 hours for the update to be applied
5. Verify by checking if **MCP Servers** appears in the left menu under APIs

### Important Configuration Note

**Critical:** If diagnostic logging is enabled via Application Insights or Azure Monitor at the global scope (All APIs), you must set the "Number of payload bytes to log" for Frontend Response to 0.

Why: Response body logging triggers buffering, which interferes with MCP server streaming behavior and can cause tool invocation failures.

To configure:
1. Navigate to **APIs** > **All APIs** > **Settings**
2. Under **Diagnostic Logs**, find **Frontend Response**
3. Set **Number of payload bytes to log** to `0`
4. Click **Save**

### Expose API as MCP Server

1. In the Azure portal, navigate to your API Management instance
2. In the left menu, under **APIs**, select **MCP Servers** > **+ Create MCP server**
3. Select **"Expose an API as an MCP server"**
4. In **Backend MCP server**:
   - Select a managed API to expose from the dropdown
   - Select one or more API operations to expose as tools (or select all)
5. In **New MCP server**:
   - Enter a **Name** for the MCP server (e.g., `customer-api-mcp`)
   - Optionally, enter a **Description** explaining what the server provides
6. Click **Create**

The MCP server is created and listed in the **MCP Servers** blade with its Server URL endpoint in this format:
```
https://[your-apim-instance].azure-api.net/mcp/[server-name]
```

### Configure Policies

Configure API Management policies to manage the MCP server. These policies apply to all API operations exposed as tools.

**Important:** Do not access the response body using `context.Response.Body` within MCP server policies, as this triggers response buffering and interferes with streaming behavior.

To configure policies:
1. Navigate to **APIs** > **MCP Servers**
2. Select your MCP server
3. Click **Policies** in the toolbar
4. Edit the policy XML

**Example: Rate Limiting by IP**
```xml
<policies>
    <inbound>
        <base />
        <rate-limit-by-key calls="5" 
                           renewal-period="30" 
                           counter-key="@(context.Request.IpAddress)" 
                           remaining-calls-variable-name="remainingCallsPerIP" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

**Example: Add Authentication Header**
```xml
<policies>
    <inbound>
        <base />
        <set-header name="Authorization" exists-action="override">
            <value>Bearer {{api-key-secret}}</value>
        </set-header>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

### Add MCP Server in Visual Studio Code

1. Open Visual Studio Code with GitHub Copilot installed
2. Use the **"MCP: Add Server"** command from the Command Palette (Ctrl+Shift+P or Cmd+Shift+P)
3. Select server type: **HTTP (HTTP or Server Sent Events)**
4. Enter the Server URL from API Management:
```
   https://your-apim.azure-api.net/mcp/your-server-name
```
5. Enter a Server ID of your choice (e.g., `azure-customer-api`)
6. Choose where to save:
   - **Workspace settings**: `.vscode/mcp.json` (project-specific)
   - **User settings**: Global `settings.json` (available in all projects)

Add authentication configuration to the JSON file:

**Using Subscription Key:**
```json
{
  "mcp": {
    "servers": {
      "azure-customer-api": {
        "url": "https://your-apim.azure-api.net/mcp/your-server-name",
        "headers": {
          "Ocp-Apim-Subscription-Key": "your-subscription-key"
        }
      }
    }
  }
}
```

**Using OAuth Token:**
```json
{
  "mcp": {
    "servers": {
      "azure-customer-api": {
        "url": "https://your-apim.azure-api.net/mcp/your-server-name",
        "headers": {
          "Authorization": "Bearer your-oauth-token"
        }
      }
    }
  }
}
```

### Use Tools in Agent Mode

1. In GitHub Copilot chat, select **Agent mode** (click the agent icon)
2. Click the **Tools** button to see available tools from connected MCP servers
3. Select one or more tools from the MCP server
4. Enter a prompt to invoke the tool (e.g., "Get customer details for ID 12345")
5. Select **Continue** to see results from the API

### Troubleshooting

**Issue: 401 Unauthorized error**
- Solution: Add authentication using the `set-header` policy to manually attach the authorization token
- Verify the subscription key or OAuth token is valid and has access to the API
- Check that the policy is applied at the correct scope (MCP server or API level)

**Issue: API call works in API Management test console but fails in agent**
- Solution: Verify security policies are correctly configured for the MCP server
- Check that CORS policies allow requests from the MCP client
- Ensure the endpoint URL is correct and accessible from the client

**Issue: MCP server streaming fails with diagnostic logs enabled**
- Solution: Disable response body logging at the **All APIs** scope
- Navigate to **APIs** > **All APIs** > **Settings** > **Diagnostic Logs**
- Set **Number of payload bytes to log** for Frontend Response to `0`

**Issue: Tools not appearing in VS Code**
- Solution: Restart VS Code after adding the MCP server configuration
- Verify the MCP server URL is accessible by testing in a browser
- Check the GitHub Copilot extension is up to date

**Issue: "Failed to connect to MCP server"**
- Solution: Verify your Azure API Management instance is in a supported tier
- Check network connectivity from your machine to the APIM endpoint
- Ensure no firewall or proxy is blocking the connection

---

## <img src="https://a0.awsstatic.com/libra-css/images/site/fav/favicon.ico" width="24" height="24" alt="AWS" /> AWS

> **Important Note**
>
> Unlike other platform sections that show how to **create MCP servers from existing APIs**, this AWS section covers the **MCP Client Proxy** for connecting to **existing MCP servers already deployed on AWS**. It handles SigV4 authentication but does not create MCP servers.
>
> **This section assumes you already have an MCP-compliant server deployed on AWS.** If you need to create an MCP server from existing AWS services, you'll need to implement the MCP protocol in your Lambda/API Gateway first.

This guide is based on the [MCP Proxy for AWS](https://github.com/aws/mcp-proxy-for-aws) repository. The MCP Proxy for AWS is a lightweight client-side bridge between MCP clients (like Claude Desktop, Amazon Q Developer CLI) and IAM-secured MCP servers on AWS that use SigV4 authentication.

### Overview

The MCP Proxy for AWS solves a key integration challenge: while the official MCP specification supports OAuth-based authentication, MCP servers on AWS can use AWS IAM authentication (SigV4). Standard MCP clients don't natively support SigV4 request signing.

This package bridges that gap by:
- Handling SigV4 authentication automatically using your local AWS credentials
- Providing seamless integration with existing MCP clients
- Eliminating the need to build custom MCP clients with SigV4 signing logic

### Prerequisites

- Python 3.10 or later
- `uv` package manager installed ([installation guide](https://github.com/astral-sh/uv))
- AWS CLI installed and configured with valid credentials
- AWS account with appropriate IAM permissions
- An MCP server endpoint on AWS (e.g., Amazon Bedrock AgentCore, API Gateway, Lambda Function URL)
- Docker Desktop (optional, for containerized deployment)

### Installation

The MCP Proxy for AWS can be used in two ways:
1. **As a proxy** - Bridge between MCP clients and AWS MCP servers (covered in this guide)
2. **As a library** - Programmatic integration with AI frameworks like LangChain, LlamaIndex, Strands Agents (see [repository documentation](https://github.com/aws/mcp-proxy-for-aws))

**Option 1: Using PyPI (Recommended)**
```bash
uvx mcp-proxy-for-aws@latest https://your-mcp-endpoint.execute-api.us-east-1.amazonaws.com
```

**Option 2: Using Local Repository**
```bash
git clone https://github.com/aws/mcp-proxy-for-aws.git
cd mcp-proxy-for-aws
uv run mcp_proxy_for_aws/server.py https://your-mcp-endpoint.execute-api.us-east-1.amazonaws.com
```

**Option 3: Using Docker**
```bash
# Build the image
docker build -t mcp-proxy-for-aws .

# Run the container
docker run --rm \
  -v ~/.aws:/app/.aws:ro \
  -e AWS_PROFILE=default \
  -e AWS_REGION=us-east-1 \
  mcp-proxy-for-aws \
  https://your-mcp-endpoint.execute-api.us-east-1.amazonaws.com
```

**Note:** When using with MCP clients like Claude Desktop or Amazon Q Developer CLI, you don't need to run these commands manually. The client will start the proxy automatically based on your configuration.

### Endpoint URL Format

AWS MCP endpoints follow these patterns:

- **API Gateway**: `https://[api-id].execute-api.[region].amazonaws.com/[stage]/mcp`
- **Lambda Function URL**: `https://[function-url-id].lambda-url.[region].on.aws/`
- **Application Load Balancer**: `https://[alb-dns-name]/mcp`

Example:
```bash
uvx mcp-proxy-for-aws@latest https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/mcp
```

### Configuration Parameters

| Parameter | Required | Description | Default |
|-----------|----------|-------------|---------|
| `endpoint` | Yes | MCP endpoint URL | - |
| `--service` | No | AWS service name for SigV4 signing | Inferred from endpoint |
| `--profile` | No | AWS profile for credentials | `AWS_PROFILE` env var |
| `--region` | No | AWS region | `AWS_REGION` env var or `us-east-1` |
| `--read-only` | No | Disable tools requiring write permissions | `false` |
| `--retries` | No | Number of retries for upstream service calls | `0` |
| `--log-level` | No | Logging level | `INFO` |
| `--timeout` | No | Timeout in seconds for all operations | `180` |
| `--connect-timeout` | No | Connect timeout in seconds | `60` |
| `--read-timeout` | No | Read timeout in seconds | `120` |
| `--write-timeout` | No | Write timeout in seconds | `180` |

### Environment Variables

Configure AWS credentials using one of these methods:

**Method 1: Using AWS Profile**
```bash
export AWS_PROFILE=your-profile-name
export AWS_REGION=us-east-1
```

**Method 2: Using Access Keys**
```bash
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_SESSION_TOKEN=your-session-token  # Only for temporary credentials
export AWS_REGION=us-east-1
```

### MCP Client Configuration

**For Amazon Q Developer CLI**

Edit `~/.aws/amazonq/mcp.json`:

Using uv:
```json
{
  "mcpServers": {
    "aws-mcp-server": {
      "disabled": false,
      "type": "stdio",
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/mcp",
        "--profile",
        "default",
        "--region",
        "us-east-1",
        "--log-level",
        "INFO"
      ]
    }
  }
}
```

Using Docker:
```json
{
  "mcpServers": {
    "aws-mcp-server": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "--volume",
        "/Users/yourname/.aws:/app/.aws:ro",
        "-e",
        "AWS_PROFILE=default",
        "-e",
        "AWS_REGION=us-east-1",
        "mcp-proxy-for-aws",
        "https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/mcp"
      ],
      "env": {}
    }
  }
}
```

**For Claude Desktop**

Edit configuration file:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json` (typically `C:\Users\YourUsername\AppData\Roaming\Claude\claude_desktop_config.json`)

Using uvx:
```json
{
  "mcpServers": {
    "aws-api": {
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/mcp",
        "--profile",
        "default",
        "--region",
        "us-east-1"
      ]
    }
  }
}
```

Using Docker (Windows example):
```json
{
  "mcpServers": {
    "aws-api": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "--volume",
        "C:\\Users\\YourUsername\\.aws:/app/.aws:ro",
        "-e",
        "AWS_PROFILE=default",
        "-e",
        "AWS_REGION=us-east-1",
        "mcp-proxy-for-aws",
        "https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/mcp"
      ],
      "env": {}
    }
  }
}
```

**Important:** After updating the configuration, completely quit Claude Desktop and restart it.

### Additional Resources

- [MCP Proxy for AWS Repository](https://github.com/aws/mcp-proxy-for-aws)
- [Programmatic Access Documentation](https://github.com/aws/mcp-proxy-for-aws#programmatic-access)
- [AWS Documentation](https://docs.aws.amazon.com/)

### Troubleshooting

**Issue: "Unable to locate credentials"**
- Solution: Verify AWS CLI is configured with `aws configure` or environment variables are set correctly
- Check credential precedence: environment variables > profile > IAM role
- Test AWS credentials with: `aws sts get-caller-identity`

**Issue: "Access Denied" or 403 errors**
- Solution: Verify IAM user/role has permissions to invoke the MCP endpoint
- Check the SigV4 service name matches the AWS service hosting your endpoint
- Ensure the `--service` parameter is correct (e.g., `execute-api` for API Gateway, `lambda` for Lambda Function URLs)

**Issue: Connection timeout**
- Solution: Increase timeout values with `--timeout`, `--connect-timeout`, or `--read-timeout` flags
- Verify network connectivity to the AWS endpoint
- Check if VPN or firewall is blocking the connection

**Issue: "Invalid endpoint URL"**
- Solution: Ensure endpoint URL includes the full path including protocol (`https://`)
- Verify the endpoint is accessible and returns valid MCP responses
- Test the endpoint with `curl` to confirm it's reachable

**Issue: MCP server not starting in Claude Desktop**
- Solution: Verify `uv` is installed and accessible from command line (`uv --version`)
- Check Claude Desktop logs for error messages
- Ensure the endpoint URL is correct and accessible from your machine

For more troubleshooting help, see the [repository's documentation](https://github.com/aws/mcp-proxy-for-aws)

---

## Security Best Practices

### Credential Management

1. **Never commit credentials to version control**
   - Use `.env` files and add them to `.gitignore`
   - Use environment variables for sensitive data
   - Rotate API keys and secrets regularly

2. **Use least-privilege access**
   - Grant only the minimum permissions required for each MCP server
   - Create dedicated service accounts or IAM roles
   - Regularly audit and review permissions

3. **Secure credential storage**
   - AWS: Use AWS Secrets Manager or Systems Manager Parameter Store
   - Azure: Use Azure Key Vault
   - GCP: Use Secret Manager
   - Local development: Use secure credential managers like 1Password or LastPass

### Network Security

1. **Use HTTPS exclusively**
   - Never expose MCP servers over unencrypted HTTP
   - Validate SSL/TLS certificates
   - Use strong cipher suites

2. **Implement rate limiting**
   - Protect against abuse and DDoS attacks
   - Configure appropriate limits based on expected usage
   - Monitor for unusual traffic patterns

3. **Restrict network access**
   - Use firewall rules to limit access to trusted IP ranges
   - Implement VPN or private networking where possible
   - Use API gateways for additional security layers

### Authentication and Authorization

1. **Use strong authentication methods**
   - OAuth 2.0 for user-facing applications
   - API keys with proper rotation policies
   - Service-to-service authentication with short-lived tokens

2. **Implement proper authorization**
   - Validate permissions for each API operation
   - Use role-based access control (RBAC)
   - Log all access attempts for audit purposes

### Monitoring and Logging

1. **Enable comprehensive logging**
   - Log all API requests and responses (excluding sensitive data)
   - Monitor for failed authentication attempts
   - Set up alerts for suspicious activities

2. **Regular security audits**
   - Review access logs periodically
   - Conduct security assessments of MCP server configurations
   - Keep dependencies up to date with security patches

---

## Troubleshooting Common Issues

### Connection Issues

**Problem: Cannot connect to MCP server**

Checklist:
- Verify the endpoint URL is correct and accessible
- Check network connectivity with `curl` or `telnet`
- Ensure firewall rules allow outbound connections
- Verify DNS resolution for the endpoint domain
- Test with verbose logging enabled (`--log-level DEBUG` for AWS)

### Authentication Failures

**Problem: 401 Unauthorized or 403 Forbidden errors**

Checklist:
- Verify credentials are correctly configured
- Check credential expiration (especially for temporary tokens)
- Ensure the service account has necessary permissions
- Validate API key or subscription key is active
- Test authentication separately from MCP connection

### Performance Issues

**Problem: Slow response times or timeouts**

Checklist:
- Check network latency to the endpoint
- Increase timeout values in configuration
- Monitor API backend performance
- Review rate limiting settings
- Consider caching strategies for frequently accessed data

### Tool Discovery Issues

**Problem: Tools not appearing in MCP client**

Checklist:
- Restart the MCP client application
- Verify MCP server is running and accessible
- Check client configuration file for syntax errors
- Ensure the MCP server properly exposes tools in its schema
- Review client logs for error messages

### Configuration Issues

**Problem: Invalid configuration errors**

Checklist:
- Validate JSON syntax in configuration files
- Check for missing required parameters
- Verify file paths are absolute and correct
- Ensure environment variables are properly set
- Review configuration against platform-specific documentation
