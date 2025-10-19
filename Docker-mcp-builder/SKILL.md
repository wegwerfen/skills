---
name: docker-mcp-builder
description: Guide for creating MCP (Model Context Protocol) servers that run in Docker containers using the Docker MCP Gateway. Supports both Python (FastMCP) and Node.js (TypeScript) implementations for building secure, containerized MCP servers.
license: Complete terms in LICENSE.txt
---

# Docker MCP Server Development Guide

## Overview

To create MCP (Model Context Protocol) servers that run in Docker containers and integrate with Claude Desktop via the Docker MCP Gateway, use this skill. This guide supports both **Python (FastMCP)** and **Node.js (TypeScript)** implementations, enabling you to build secure, containerized MCP servers with proper tooling and deployment workflows.

---

# Process

## 🚀 High-Level Workflow

Creating a Docker-based MCP server involves four main phases:

### Phase 1: Planning and Requirements Gathering

#### 1.1 Gather Initial Requirements

Before generating the MCP server, collect the following information from the user:

**Language Selection:**
Ask: "Would you like to create this MCP server in Python or Node.js?"
- **Python**: Uses FastMCP framework, simpler for quick implementations
- **Node.js**: Uses TypeScript and MCP SDK, better for complex projects with strict typing

**Service/Tool Information:**
- **Service Name**: What service or functionality will this MCP server provide?
- **API Documentation**: If integrating with an API, what is the documentation URL?
- **Required Features**: List the specific features/tools to implement
- **Authentication**: Does this require API keys, OAuth, or other authentication?
- **Data Sources**: Will this access files, databases, APIs, or other data sources?

**Special Docker Requirements:**
- Security tools needed (e.g., nmap, nikto for pentesting servers)?
- System packages required in the Docker container?
- Network capabilities needed?
- Volume mounts required?

**If any critical information is missing, ASK THE USER before proceeding.**

#### 1.2 Understand Docker MCP Gateway Architecture

The Docker MCP Gateway provides:
- Containerized MCP servers with isolation and security
- Docker Desktop secrets integration for API keys
- Custom catalog system for server management
- stdio transport for Claude Desktop integration

**Key Components:**
```
Claude Desktop → Docker MCP Gateway → Containerized MCP Server → External API/Service
                        ↓
                Docker Desktop Secrets
```

#### 1.3 Review API Documentation (if applicable)

If integrating with an external API, use web search and WebFetch to load:
- Official API reference documentation
- Authentication and authorization requirements
- Rate limiting and pagination patterns
- Error responses and status codes
- Available endpoints and their parameters

---

### Phase 2: Implementation

#### 2.1 Understand Language-Specific Critical Rules

**For Python (FastMCP):**

Critical restrictions that prevent runtime errors:
1. **NO `@mcp.prompt()` decorators** - They break Claude Desktop
2. **NO `prompt` parameter to FastMCP()** - It breaks Claude Desktop
3. **NO type hints from typing module** - No `Optional`, `Union`, `List[str]`, etc.
4. **NO complex parameter types** - Use `param: str = ""` not `param: str = None`
5. **SINGLE-LINE DOCSTRINGS ONLY** - Multi-line docstrings cause gateway panic errors
6. **DEFAULT TO EMPTY STRINGS** - Use `param: str = ""` never `param: str = None`

**For Node.js (TypeScript):**

Critical requirements for proper operation:
1. **Use MCP TypeScript SDK** - Import from `@modelcontextprotocol/sdk`
2. **Strict TypeScript mode** - Enable in tsconfig.json
3. **Zod schemas required** - All input validation uses Zod with `.strict()`
4. **Explicit type annotations** - All parameters and return types typed
5. **NO `any` types** - Use `unknown` or proper types
6. **Build before running** - Must run `npm run build` successfully

#### 2.2 Universal Docker Requirements

Regardless of language, all implementations must:
1. **ALWAYS use Docker** - The server must run in a Docker container
2. **ALWAYS log to stderr** - Use proper logging configuration
3. **ALWAYS handle errors gracefully** - Return user-friendly error messages
4. **ALWAYS return strings from tools** - All tools must return formatted strings
5. **Create non-root user** - Run containers as non-root for security
6. **Use environment variables** - For API keys and configuration
7. **Server must expose tools via stdio transport**
8. **Container must be built and tagged properly**

#### 2.3 Implement Tools Following Language Patterns

**Python Tool Pattern:**
```python
@mcp.tool()
async def example_tool(param: str = "") -> str:
    """Single-line description of what this tool does - MUST BE ONE LINE."""
    logger.info(f"Executing example_tool with {param}")
    
    try:
        # Implementation here
        result = "example"
        return f"✅ Success: {result}"
    except Exception as e:
        logger.error(f"Error: {e}")
        return f"❌ Error: {str(e)}"
```

**Node.js Tool Pattern:**
```typescript
import { z } from "zod";

const ExampleInputSchema = z.object({
  param: z.string()
    .min(1, "Parameter is required")
    .describe("Description of parameter")
}).strict();

type ExampleInput = z.infer<typeof ExampleInputSchema>;

server.registerTool(
  "example_tool",
  {
    title: "Example Tool",
    description: `Single-line summary.

Detailed explanation.

Args:
  - param (string): Description

Returns:
  Success message or error details`,
    inputSchema: ExampleInputSchema,
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async (params: ExampleInput) => {
    try {
      return {
        content: [{
          type: "text",
          text: "✅ Success: result"
        }]
      };
    } catch (error) {
      return {
        content: [{
          type: "text",
          text: `❌ Error: ${error instanceof Error ? error.message : String(error)}`
        }]
      };
    }
  }
);
```

#### 2.4 Create Required Files

Your response must be organized in **TWO distinct sections**:

**SECTION 1: FILES TO CREATE**
Generate all necessary files with complete content that the user can copy and save.
DO NOT create duplicate files or variations. Each file appears ONCE with complete content.

**SECTION 2: INSTALLATION INSTRUCTIONS**
Provide step-by-step commands the user needs to run on their computer.
Present as a clean, numbered list without duplicate instruction sets.

---

### Phase 3: File Templates and Patterns

#### 3.1 Python File Templates

**Dockerfile (Python):**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
ENV PYTHONUNBUFFERED=1

# Install system packages if needed
# RUN apt-get update && apt-get install -y package-name && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY [SERVER_NAME]_server.py .

RUN useradd -m -u 1000 mcpuser && chown -R mcpuser:mcpuser /app
USER mcpuser

CMD ["python", "[SERVER_NAME]_server.py"]
```

**requirements.txt:**
```
mcp[cli]>=1.2.0
httpx
# Add any other required libraries
```

**Server Structure ([SERVER_NAME]_server.py):**
```python
#!/usr/bin/env python3
"""Simple [SERVICE_NAME] MCP Server - [DESCRIPTION]"""
import os, sys, logging
from datetime import datetime, timezone
import httpx
from mcp.server.fastmcp import FastMCP

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', stream=sys.stderr)
logger = logging.getLogger("[SERVER_NAME]-server")

mcp = FastMCP("[SERVER_NAME]")

# Configuration
# API_TOKEN = os.environ.get("[SERVER_NAME_UPPER]_API_TOKEN", "")

# Tools implementation here

if __name__ == "__main__":
    logger.info("Starting [SERVICE_NAME] MCP server...")
    try:
        mcp.run(transport='stdio')
    except Exception as e:
        logger.error(f"Server error: {e}", exc_info=True)
        sys.exit(1)
```

#### 3.2 Node.js File Templates

**Dockerfile (Node.js):**
```dockerfile
FROM node:20-slim
WORKDIR /app

COPY package*.json ./
COPY tsconfig.json ./
RUN npm ci --only=production

COPY src/ ./src/
RUN npm run build

RUN useradd -m -u 1000 mcpuser && chown -R mcpuser:mcpuser /app
USER mcpuser

CMD ["node", "dist/index.js"]
```

**package.json:**
```json
{
  "name": "[server-name]-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.4",
    "axios": "^1.7.9",
    "zod": "^3.24.1"
  },
  "devDependencies": {
    "@types/node": "^22.10.2",
    "typescript": "^5.7.2"
  }
}
```

**tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

**Server Structure (src/index.ts):**
```typescript
#!/usr/bin/env node
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import axios from "axios";

const server = new McpServer({ name: "[server-name]-mcp", version: "1.0.0" });

// Constants and configuration
const API_KEY = process.env.API_KEY || "";

// Tool implementations here

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("[SERVICE_NAME] MCP server running via stdio");
}

main().catch((error) => {
  console.error("Server error:", error);
  process.exit(1);
});
```

#### 3.3 Documentation Templates

**readme.txt:**
Create comprehensive documentation including:
- Purpose and features
- Prerequisites
- Installation instructions
- Usage examples
- Architecture diagram
- Troubleshooting guide
- Security considerations

**CLAUDE.md:**
Create implementation guide including:
- Overview and tech stack
- Tool descriptions with parameters
- Environment variables
- API integration details
- Security notes
- Development notes (language-specific)

---

### Phase 4: Deployment and Installation

#### 4.1 Build and Test

**For Python:**
```bash
# Verify syntax
python -m py_compile [SERVER_NAME]_server.py

# Build Docker image
docker build -t [SERVER_NAME]-mcp-server .
```

**For Node.js:**
```bash
# Build TypeScript
npm run build

# Verify dist/index.js exists
ls -la dist/

# Build Docker image
docker build -t [SERVER_NAME]-mcp-server .
```

#### 4.2 Configure Docker MCP Gateway

**Set up secrets (if needed):**
```bash
docker mcp secret set [SECRET_NAME]="your-secret-value"
docker mcp secret list
```

**Create custom catalog:**
```bash
mkdir -p ~/.docker/mcp/catalogs
nano ~/.docker/mcp/catalogs/custom.yaml
```

**Add catalog entry:**
```yaml
version: 2
name: custom
displayName: Custom MCP Servers
registry:
  [SERVER_NAME]:
    description: "[DESCRIPTION]"
    title: "[SERVICE_NAME]"
    type: server
    dateAdded: "[CURRENT_DATE]"  # ISO 8601 format
    image: [SERVER_NAME]-mcp-server:latest
    ref: ""
    tools:
      - [tool_name_1]
      - [tool_name_2]
    # If using secrets:
    # secrets:
    #   - [SECRET_NAME]
```

**Enable server:**
```bash
docker mcp server enable custom
docker mcp server enable custom.[SERVER_NAME]
docker mcp server list
```

#### 4.3 Configure Claude Desktop

Edit configuration file:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "docker-mcp": {
      "command": "docker",
      "args": [
        "mcp",
        "run",
        "--catalogs",
        "custom",
        "[YOUR_HOME]/.docker/mcp/catalogs/custom.yaml"
      ]
    }
  }
}
```

Replace `[YOUR_HOME]` with your home directory path.

**Restart Claude Desktop** to load the new server.

---

# Implementation Patterns

## Output Formatting Guidelines

Use emojis for visual clarity:
- ✅ Success operations
- ❌ Errors or failures
- ⏱️ Time-related information
- 📊 Data or statistics
- 🔍 Search or lookup operations
- ⚡ Actions or commands
- 🔒 Security-related information
- 📁 File operations
- 🌐 Network operations
- ⚠️ Warnings

**Python formatting:**
```python
return f"""📊 Results:
- Field 1: {value1}
- Field 2: {value2}

Summary: {summary}"""
```

**Node.js formatting:**
```typescript
const lines = ["📊 Results:", `- Field 1: ${value1}`, `- Field 2: ${value2}`, "", `Summary: ${summary}`];
return { content: [{ type: "text", text: lines.join("\n") }] };
```

## API Integration Patterns

**Python with httpx:**
```python
async with httpx.AsyncClient() as client:
    try:
        response = await client.get(url, headers=headers, timeout=10)
        response.raise_for_status()
        data = response.json()
        return f"✅ Result: {formatted_data}"
    except httpx.HTTPStatusError as e:
        return f"❌ API Error: {e.response.status_code}"
    except Exception as e:
        return f"❌ Error: {str(e)}"
```

**Node.js with axios:**
```typescript
async function makeApiRequest<T>(endpoint: string): Promise<T> {
  try {
    const response = await axios.get<T>(`${API_BASE_URL}/${endpoint}`, {
      headers: { "Authorization": `Bearer ${API_KEY}` },
      timeout: 10000
    });
    return response.data;
  } catch (error) {
    if (axios.isAxiosError(error)) {
      if (error.response?.status === 429) {
        throw new Error("Rate limit exceeded");
      }
      throw new Error(error.response?.data?.message || error.message);
    }
    throw error;
  }
}
```

---

# Quality Checklist

Before finalizing your Docker MCP server implementation, verify:

## Universal Checks
- [ ] All necessary files created with proper naming
- [ ] All placeholders replaced with actual values
- [ ] Usage examples provided
- [ ] Security handled via Docker secrets
- [ ] Catalog properly formatted with version: 2
- [ ] Date format is ISO 8601 (YYYY-MM-DDTHH:MM:SSZ)
- [ ] Claude config JSON has no comments
- [ ] Each file appears exactly once
- [ ] Instructions are clear and numbered
- [ ] Error handling in every tool
- [ ] Clear separation between files and user instructions
- [ ] All tools return strings/content objects
- [ ] Logging configured to stderr
- [ ] Non-root user in Dockerfile
- [ ] Environment variables for secrets

## Python-Specific Checks
- [ ] No @mcp.prompt() decorators used
- [ ] No prompt parameter in FastMCP()
- [ ] No complex type hints from typing module
- [ ] ALL tool docstrings are SINGLE-LINE only
- [ ] ALL parameters default to empty strings ("") not None
- [ ] Check for empty strings with .strip() not just truthiness
- [ ] Proper imports: httpx, logging, mcp.server.fastmcp
- [ ] Server runs with mcp.run(transport='stdio')

## Node.js-Specific Checks
- [ ] All tools registered using registerTool with complete configuration
- [ ] All tools use Zod schemas with .strict()
- [ ] TypeScript strict mode enabled in tsconfig.json
- [ ] No any types used
- [ ] Explicit Promise<T> return types
- [ ] Build script configured (npm run build)
- [ ] Package.json includes all dependencies
- [ ] Main entry point is dist/index.js
- [ ] Server name follows format: {service}-mcp-server
- [ ] npm run build completes successfully

## Docker Checks
- [ ] Dockerfile uses appropriate base image
- [ ] Dependencies installed efficiently
- [ ] Non-root user created (mcpuser)
- [ ] Container runs as non-root
- [ ] Working directory set to /app
- [ ] CMD properly configured for stdio transport
- [ ] Image builds successfully
- [ ] Image tagged correctly

## Deployment Checks
- [ ] Custom catalog created in ~/.docker/mcp/catalogs/
- [ ] Server enabled in Docker MCP
- [ ] Claude Desktop config updated
- [ ] All secrets configured (if needed)
- [ ] Server appears in docker mcp server list
- [ ] Tools appear in Claude Desktop after restart
