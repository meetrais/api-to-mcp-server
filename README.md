# Exposing Existing APIs as MCP Servers

This repository provides comprehensive documentation on exposing existing APIs as Model Context Protocol (MCP) servers across different platforms and tools. MCP servers enable AI assistants to securely interact with your APIs through a standardized protocol.

## Quick Start Guide

Choose your platform based on your existing infrastructure:

| Platform | Prerequisites | Setup Time |
|----------|---------------|------------|
| [Postman](#postman) | Node.js 18+ | 10 minutes |
| [Google Cloud (Apigee)](#google-cloud-platform-apigee) | GCP Project, Apigee X | 30 minutes |
| [Microsoft Azure](#microsoft-azure) | Azure APIM instance | 20 minutes |
| [AWS](#aws) | AWS account with Bedrock AgentCore, AWS CLI | 20 minutes |

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

Amazon Bedrock AgentCore Gateway provides **native MCP support** that allows enterprises to turn their existing APIs, Lambda functions, and other backend services into secure, governed MCP tools through a fully managed gateway — without deploying separate MCP servers.

> **Native MCP Support**
>
> Amazon Bedrock AgentCore Gateway acts as a fully managed "tool front door" for your AI agents. It aggregates multiple backend systems — including REST APIs, AWS Lambda functions, Smithy models, and existing MCP servers — into a single, unified MCP-compatible endpoint. You don't need to write MCP protocol code or manage MCP server infrastructure.

### Key Features

- **No MCP server code required**: Transform existing REST APIs, Lambda functions, and Smithy models into MCP tools using your existing API specifications
- **Fully managed infrastructure**: AgentCore Gateway handles MCP servers, protocol transcoding, tool discovery, and routing
- **Multiple target types**: Supports Lambda functions, OpenAPI schemas, Smithy models, native MCP servers, and built-in integration templates (Salesforce, Slack, Jira, Asana, Zendesk)
- **Enterprise-grade security**: Dual authentication model with inbound (agent-to-gateway) and outbound (gateway-to-backend) authorization, including IAM (SigV4) and OAuth support
- **Automatic tool discovery**: Agents dynamically discover available tools at runtime via the standard MCP `tools/list` interface
- **Stateful MCP support**: AgentCore Runtime supports advanced stateful MCP features including elicitation (server-initiated multi-turn conversations), sampling, and real-time progress notifications
- **Framework compatibility**: Works with Strands Agents, LangChain, LlamaIndex, and other popular agent frameworks

### Key Benefits

1. **No Added Operational Burden**: You don't need to build, deploy, or manage MCP servers for each of your APIs. Create a gateway, add your targets, and AgentCore handles the rest — fully managing MCP protocol handling, transcoding, and tool routing.

2. **Unified Tool Interface**: Aggregate tools from multiple backend sources (REST APIs, Lambda, existing MCP servers) behind a single MCP endpoint, simplifying agent development and reducing integration complexity.

3. **Comprehensive Tool Security**: AgentCore ensures all agentic interactions are secure:
   - **Inbound Authorization**: Control which agents and clients can access your gateway using Amazon Cognito OAuth or IAM (SigV4) authentication
   - **Outbound Authorization**: Manage how the gateway authenticates with each backend service (API keys, OAuth, IAM roles)
   - Store sensitive credentials in [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) or [Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
   - Enforce least-privilege IAM permissions for agents and users

4. **Centralized Tool Catalog**: The gateway provides a stable entry point for agents to discover and search for available tools at scale. Tools are indexed and discoverable via the standard MCP protocol.

### Supported Target Types

| Target Type | Description | Use Case |
|-------------|-------------|----------|
| **AWS Lambda** | Execute custom business logic via Lambda functions | Custom tool implementations, serverless backends |
| **OpenAPI Schema** | Convert REST APIs into MCP tools using OpenAPI 3.0/3.1 specs | Existing REST APIs, third-party services |
| **Smithy Model** | Define structured API interfaces using Smithy IDL | AWS service integrations, strongly typed APIs |
| **MCP Server** | Incorporate tools from existing MCP servers as native targets | Pre-built MCP servers, third-party MCP tools |
| **Built-in Templates** | Pre-configured integration templates | Salesforce, Slack, Jira, Asana, Zendesk |

### How It Works

1. **Create a Gateway**: Use the AWS Console, AgentCore CLI, or the `CreateGateway` API to create a gateway endpoint that serves as the unified MCP interface for your agents.

2. **Add Targets**: Configure one or more targets on your gateway. Each target maps a backend service (Lambda, REST API, MCP server) to the gateway, defining how requests are routed and authenticated.

3. **Tool Indexing**: The gateway uses the `SynchronizeGatewayTargets` API to perform protocol handshakes and index available tools from each target. This can be implicit (automatic) or explicit (manual refresh).

4. **Connect Agents**: Point your AI agents to the gateway's MCP endpoint. Agents discover tools dynamically via `tools/list` and invoke them via `tools/call` — all through the standard MCP protocol.

### Prerequisites

- AWS account with access to Amazon Bedrock AgentCore
- AWS CLI installed and configured with appropriate permissions
- Python 3.11+ or Node.js 20+ (for server development)
- Docker installed (for containerized deployments to AgentCore Runtime)
- Existing backend services to expose as tools:
  - REST APIs with OpenAPI 3.0/3.1 specifications, OR
  - AWS Lambda functions with tool schemas, OR
  - Existing MCP servers with Streamable HTTP transport

### Getting Started

**1. Create a Gateway**

Using the AWS CLI:
```bash
aws bedrock-agentcore-control create-gateway \
  --name "my-api-gateway" \
  --description "MCP gateway for my existing APIs"
```

Or use the AWS Console to create a gateway through the graphical interface, which allows you to configure authorization, define the gateway, and add targets in one step.

**2. Add Targets**

Add your existing APIs as targets on the gateway. The configuration depends on your target type:

**For REST APIs (OpenAPI):**
- Prepare an OpenAPI 3.0 or 3.1 specification file describing your API
- Upload the spec to an S3 bucket or provide it inline
- The `operationId` in the specification becomes the MCP tool name

**For Lambda Functions:**
- Provide the Lambda function ARN
- Define the tool schema describing inputs and outputs

**For Existing MCP Servers:**
- Provide the MCP server endpoint URL
- The gateway performs a protocol handshake to index available tools

**3. Configure Authentication**

Set up the dual authentication model:

**Inbound (Agent-to-Gateway):**
- Configure an OAuth authorizer (e.g., Amazon Cognito) for production use
- Alternatively use IAM (SigV4) for service-to-service communication

**Outbound (Gateway-to-Backend):**
- Configure credentials for each target (API keys, OAuth tokens, IAM roles)
- Store secrets in AWS Secrets Manager or Systems Manager Parameter Store

**4. Synchronize and Verify**

Synchronize gateway targets to index available tools:
```bash
aws bedrock-agentcore-control synchronize-gateway-targets \
  --gateway-id "your-gateway-id"
```

**5. Connect Your Agent**

Once the gateway is live, it provides a managed MCP endpoint URL. Connect your agent to this URL and it will dynamically discover all available tools.

### Deploy MCP Servers in AgentCore Runtime

If you need to host custom MCP servers, AgentCore Runtime provides a fully managed serverless hosting environment:

**1. Containerize Your MCP Server:**
- Ensure your server supports Streamable HTTP transport
- Configure it to listen on `0.0.0.0:8000/mcp` (the default AgentCore endpoint)

**2. Push to Amazon ECR:**
```bash
# Create an ECR repository
aws ecr create-repository --repository-name my-mcp-server

# Build, tag, and push your container image
docker build -t my-mcp-server .
docker tag my-mcp-server:latest <account_id>.dkr.ecr.<region>.amazonaws.com/my-mcp-server:latest
docker push <account_id>.dkr.ecr.<region>.amazonaws.com/my-mcp-server:latest
```

**3. Deploy to AgentCore Runtime:**
```bash
aws bedrock-agentcore-control create-agent-runtime \
  --agent-runtime-name "my-mcp-runtime" \
  --agent-runtime-artifact '{ "containerConfiguration": { "containerUri": "<your-image-uri>" } }' \
  --role-arn "arn:aws:iam::<account_id>:role/AgentCoreExecutionRole"
```

**4. Register as Gateway Target:**

Add the deployed runtime as a target in your AgentCore Gateway for centralized management and tool discovery.

### MCP Client Configuration

**For Amazon Q Developer CLI**

Edit `~/.aws/amazonq/mcp.json`:

```json
{
  "mcpServers": {
    "agentcore-gateway": {
      "type": "http",
      "url": "https://your-agentcore-gateway-endpoint-url/mcp",
      "env": {
        "AWS_REGION": "us-east-1",
        "AUTH_TOKEN": "your-cognito-or-iam-token"
      }
    }
  }
}
```

**For Claude Desktop**

Edit configuration file:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "agentcore-gateway": {
      "type": "http",
      "url": "https://your-agentcore-gateway-endpoint-url/mcp",
      "env": {
        "AWS_REGION": "us-east-1",
        "AUTH_TOKEN": "your-cognito-or-iam-token"
      }
    }
  }
}
```

**Important:** After updating the configuration, completely quit your MCP client and restart it.

> **Note**: If your AgentCore Gateway uses IAM (SigV4) authentication, you can use the [MCP Proxy for AWS](https://github.com/aws/mcp-proxy-for-aws) as a client-side bridge that automatically handles SigV4 request signing. See the supplementary section below.

---

### Supplementary: MCP Proxy for AWS (SigV4 Bridge)

The [MCP Proxy for AWS](https://github.com/aws/mcp-proxy-for-aws) is a lightweight client-side bridge between MCP clients (like Claude Desktop, Amazon Q Developer CLI) and IAM-secured MCP servers on AWS that use SigV4 authentication. Use this when your AgentCore Gateway or other AWS-hosted MCP server requires SigV4 signing.

**Prerequisites:**
- Python 3.10 or later
- `uv` package manager installed ([installation guide](https://github.com/astral-sh/uv))
- AWS CLI installed and configured with valid credentials

**Quick Start:**
```bash
uvx mcp-proxy-for-aws@latest https://your-agentcore-gateway-endpoint-url/mcp
```

**Client Configuration with SigV4 Proxy (Amazon Q Developer CLI):**
```json
{
  "mcpServers": {
    "aws-mcp-server": {
      "disabled": false,
      "type": "stdio",
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://your-agentcore-gateway-endpoint-url/mcp",
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

**Client Configuration with SigV4 Proxy (Claude Desktop):**
```json
{
  "mcpServers": {
    "aws-api": {
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://your-agentcore-gateway-endpoint-url/mcp",
        "--profile",
        "default",
        "--region",
        "us-east-1"
      ]
    }
  }
}
```

For additional proxy options (Docker, configuration parameters, environment variables), see the [MCP Proxy for AWS Repository](https://github.com/aws/mcp-proxy-for-aws).

### Additional Resources

- [Amazon Bedrock AgentCore Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore-landing.html)
- [AgentCore Gateway Developer Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore-gateway-overview.html)
- [Deploy MCP Servers in AgentCore Runtime](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore-runtime.html)
- [AgentCore GitHub Samples](https://github.com/awslabs/amazon-bedrock-agentcore-samples)
- [MCP Proxy for AWS Repository](https://github.com/aws/mcp-proxy-for-aws)
- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)

### Troubleshooting

**Issue: "Failed to create gateway" or permission errors**
- Solution: Verify your AWS account has access to Amazon Bedrock AgentCore
- Ensure your IAM user/role has the necessary `bedrock-agentcore-control:*` permissions
- Check that the service is available in your chosen AWS region

**Issue: Tool discovery failures after adding targets**
- Solution: Run `SynchronizeGatewayTargets` to refresh the tool index
- Verify your OpenAPI specification is valid OpenAPI 3.0 or 3.1 (Swagger 2.0 is not supported)
- Ensure Lambda functions are accessible and the tool schema is correctly defined
- For MCP server targets, verify the endpoint responds to MCP protocol handshakes

**Issue: "Unable to locate credentials" (when using MCP Proxy for AWS)**
- Solution: Verify AWS CLI is configured with `aws configure` or environment variables are set correctly
- Check credential precedence: environment variables > profile > IAM role
- Test AWS credentials with: `aws sts get-caller-identity`

**Issue: "Access Denied" or 403 errors**
- Solution: Check both inbound and outbound authorization configurations
- Verify the agent's OAuth token or IAM credentials are valid
- Ensure the gateway's IAM execution role has permissions to invoke backend targets
- For SigV4 endpoints, ensure the `--service` parameter is correct (e.g., `execute-api` for API Gateway)

**Issue: Connection timeout or 504 errors**
- Solution: AgentCore Gateway has a 5-minute timeout for invocations — ensure backend operations complete within this limit
- Verify network connectivity and check if VPC configuration allows outbound access
- For long-running tasks, consider writing results to Amazon Bedrock AgentCore Memory

**Issue: MCP server container not starting in AgentCore Runtime**
- Solution: Verify your container image listens on `0.0.0.0:8000/mcp`
- Check container health checks and port configuration
- Review AgentCore Runtime logs for detailed error messages
- Ensure the ECR image URI is correct and the execution role can pull from ECR

**Issue: "Failed to connect to MCP server" from client**
- Solution: Verify the gateway endpoint URL is accessible from your machine
- Check authentication configuration in your MCP client
- Ensure no firewall or proxy is blocking the connection
- Test the endpoint with `curl` to confirm it's reachable

For more troubleshooting help, see the [AgentCore documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agentcore-landing.html)

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
