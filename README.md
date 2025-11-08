# Exposing Existing APIs as MCP Servers

This repository provides comprehensive documentation on exposing existing APIs as Model Context Protocol (MCP) servers across different platforms and tools. MCP servers enable AI assistants to securely interact with your APIs through a standardized protocol.

## Quick Start Guide

Choose your platform based on your existing infrastructure:

| Platform | Prerequisites | Setup Time |
|----------|---------------|------------|
| [Postman](#postman) | Node.js 18+ | 10 minutes |
| [Google Cloud (Apigee)](#google-cloud-platform-apigee) | GCP Project, Apigee X | 30 minutes |
| [AWS](#aws) | Python 3.10+, AWS CLI | 15 minutes |
| [Microsoft Azure](#microsoft-azure) | Azure APIM instance | 20 minutes |

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
- [AWS](#aws)
- [Microsoft Azure](#microsoft-azure)
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

This guide is based on the [Apigee MCP Sample](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp) from Google Cloud Platform. The sample provides an MCP server implementation that dynamically discovers Apigee-managed API Products and exposes them as MCP tools for AI agents.

### Key Features

- Leverages existing Apigee-managed APIs without redevelopment
- Provides standardized discovery and invocation through MCP
- Extends Apigee's security features (OAuth 2.0, API Keys, Threat Protection) to AI agents
- Offers scalability, reliability, and observability through Apigee's infrastructure
- Integrates with Apigee API hub for API discovery

### Prerequisites

- Apigee X Organization with at least one environment
- GCP Project with the following APIs enabled:
  - Vertex AI API
  - Cloud Run API
  - Apigee API
- Apigee API hub enabled and provisioned in the same GCP project
- gcloud CLI installed and configured
- Docker installed (for building container images)

### Core Functionality

1. **API Product and Spec Discovery**: Connects to an API configured via environment variables to list API Products and OpenAPI specifications from Apigee API hub
2. **Dynamic Tool Generation**: Parses OpenAPI specifications and generates corresponding MCP tools for each operation
3. **MCP Tool Exposure**: Exposes generated tools for MCP clients to consume
4. **Secure API Execution**: Translates MCP tool calls into HTTP requests with proper authentication using OAuth 2.0

### Configuration

Configure the server using environment variables:

| Variable | Required | Description | Default | Example |
|----------|----------|-------------|---------|---------|
| `MCP_BASE_URL` | Yes | Base URL of the MCP API that lists products and specs | - | `https://api.example.com/mcp` |
| `MCP_CLIENT_ID` | Yes | Client ID for OAuth authentication | - | `abc123xyz` |
| `MCP_CLIENT_SECRET` | Yes | Client Secret for OAuth authentication | - | `secret456` |
| `MCP_MODE` | No | Transport protocol | `STDIO` | `STDIO` or `SSE` |
| `BASE_PATH` | No | Base path for SSE endpoints | `mcp-proxy` | `api/mcp` |
| `PORT` | No | Port for HTTP server in SSE mode | `3000` | `8080` |
| `MCP_CACHE_TTL` | No | Cache time-to-live in milliseconds | `300000` (5 min) | `600000` |

### Getting Started

**Clone the Repository:**
```bash
git clone https://github.com/GoogleCloudPlatform/apigee-samples.git
cd apigee-samples/apigee-mcp
```

For detailed setup instructions, refer to the [README in the repository](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp).

### Deployment

The repository includes a `deploy-all.sh` script that automates the deployment process:
```bash
# Set your GCP project
export GCP_PROJECT_ID=your-project-id
export APIGEE_ENV=your-apigee-environment

# Run deployment script
./deploy-all.sh
```

The script performs the following actions:
1. Builds container images for stub services and MCP server
2. Deploys services to Google Cloud Run
3. Configures Apigee artifacts (API Proxies, Products, Developer Apps)
4. Sets up Apigee API hub entries
5. Outputs the MCP server endpoint URL

### Manual Deployment

If you prefer manual deployment:
```bash
# Build the container
gcloud builds submit --tag gcr.io/${GCP_PROJECT_ID}/mcp-server

# Deploy to Cloud Run
gcloud run deploy mcp-server \
  --image gcr.io/${GCP_PROJECT_ID}/mcp-server \
  --platform managed \
  --region us-central1 \
  --set-env-vars MCP_BASE_URL=your-base-url,MCP_CLIENT_ID=your-client-id,MCP_CLIENT_SECRET=your-secret,MCP_MODE=SSE
```

### Additional Resources

- [Apigee MCP Sample Repository](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp)
- [Detailed Implementation Guide](https://github.com/GoogleCloudPlatform/apigee-samples/blob/main/apigee-mcp/README.md)
- [Apigee Documentation](https://cloud.google.com/apigee/docs)

### Troubleshooting

**Issue: "Failed to fetch API products"**
- Solution: Verify `MCP_BASE_URL` points to a valid Apigee API hub endpoint and credentials are correct

**Issue: Container fails to start**
- Solution: Check Cloud Run logs with `gcloud run services logs read mcp-server` and verify all required environment variables are set

**Issue: OAuth authentication failing**
- Solution: Ensure the Developer App in Apigee has the correct credentials and the API Product is associated with it

For more troubleshooting help, see the [repository's troubleshooting section](https://github.com/GoogleCloudPlatform/apigee-samples/tree/main/apigee-mcp#troubleshooting)

---

## <img src="https://a0.awsstatic.com/libra-css/images/site/fav/favicon.ico" width="24" height="24" alt="AWS" /> AWS

AWS provides the MCP Proxy for AWS, a lightweight client-side bridge between MCP clients and backend AWS MCP servers with SigV4 authentication.

### Prerequisites

- Python 3.10 or later
- `uv` package manager installed ([installation guide](https://github.com/astral-sh/uv))
- AWS CLI installed and configured with valid credentials
- AWS account with appropriate IAM permissions
- Docker Desktop (optional, for containerized deployment)

### Installation

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
- Mac: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
```json
{
  "mcpServers": {
    "aws-api": {
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/mcp",
        "--profile",
        "default"
      ]
    }
  }
}
```

### Troubleshooting

**Issue: "Unable to locate credentials"**
- Solution: Verify AWS CLI is configured with `aws configure` or environment variables are set correctly
- Check credential precedence: environment variables > profile > IAM role

**Issue: "Access Denied" or 403 errors**
- Solution: Verify IAM user/role has permissions to invoke the MCP endpoint
- Check the SigV4 service name matches the AWS service hosting your endpoint

**Issue: Connection timeout**
- Solution: Increase timeout values with `--timeout`, `--connect-timeout`, or `--read-timeout` flags
- Verify network connectivity to the AWS endpoint

**Issue: "Invalid endpoint URL"**
- Solution: Ensure endpoint URL includes the full path including protocol (`https://`)
- Verify the endpoint is accessible and returns valid MCP responses

---

## <img src="https://azure.microsoft.com/svghandler/azure/?width=24&height=24" width="24" height="24" alt="Microsoft Azure" /> Microsoft Azure

Azure API Management allows you to expose REST APIs as remote MCP servers using its built-in AI gateway capabilities.

### Prerequisites

- Azure API Management instance in one of these tiers:
  - Basic, Standard, Premium (requires AI Gateway Early Access)
  - Basic v2, Standard v2, or Premium v2
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