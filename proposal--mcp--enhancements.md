# MCP Server Enhancements Specification

**Version:** 0.2.0  
**Status:** DRAFT PROPOSAL  
**Last Updated:** 2026-01-21

## Important Note

**This is a draft proposal seeking community feedback.** We are **not** attempting to replace or modify the core [Model Context Protocol](https://modelcontextprotocol.io/) specification. MCP provides an excellent foundation for AI agent tool integration.

This document proposes **optional enhancements** that MCP servers could implement to improve discoverability, skill creation, and alternative access method discovery. These enhancements are designed to be fully backward compatible with the existing MCP specification.

**Source**: This proposal is part of the [dashdash project](https://github.com/visionik/dashdash) exploring AI agent interaction standards.

## 1. Introduction

This document specifies optional enhancements for Model Context Protocol (MCP) servers to provide AI-optimized metadata and guidance. As MCP becomes a standard for AI agent tool integration, additional metadata can help with skill creation, cross-platform discovery, and operational guidance.

### 1.1 Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 1.2 Design Priorities

The MCP enhancements MUST serve three priorities, in order:

1. **Enable creation of SKILL.md files** — The metadata MUST provide all information necessary for AI agents or developers to create skill definition files (such as those used by Claude Code, Clawdbot, or other agent frameworks). This includes server identity, triggering context, and operational guidance.

2. **Enable effective MCP server usage** — The metadata MUST document how to interact with the MCP server effectively, including tool capabilities, resource patterns, and operational constraints.

3. **Enable discovery of alternative access methods** — The metadata MUST document other ways to interact with the underlying product or service, including CLI tools, REST APIs, and web interfaces.

The metadata MAY provide additional details beyond these priorities, such as troubleshooting guides, performance characteristics, and advanced usage patterns.

## 2. Rationale

### 2.1 Foundation: What MCP Provides

The Model Context Protocol already provides:
- Standardized tool discovery via `tools/list`
- Resource discovery via `resources/list`
- Prompt templates via `prompts/list`
- JSON Schema for tool parameters
- Structured error responses

### 2.2 Problem Statement

Current MCP implementations lack:
- **Skill creation metadata** — No standard way to extract trigger phrases or descriptions suitable for SKILL.md files
- **Alternative access discovery** — No indication of CLI, API, or web alternatives
- **Operational guidance** — Rate limits, authentication details, best practices
- **Cross-platform context** — How MCP tools relate to other access methods

### 2.3 Benefits

- **Skill generation** — Enables automated or assisted creation of SKILL.md files
- **Unified discovery** — Single source for all access methods
- **Self-documenting** — Metadata travels with the MCP server
- **Backward compatible** — All enhancements are optional extensions
- **Consistent ecosystem** — Aligns with CLI `--ai-help` and web llms.txt patterns

## 3. Specification

### 3.1 Server Metadata Enhancement

#### 3.1.1 Enhanced Server Info

MCP servers SHOULD extend the `initialize` response to include enhanced metadata:

```json
{
  "protocolVersion": "2024-11-05",
  "serverInfo": {
    "name": "acme-mcp",
    "version": "1.0.0"
  },
  "capabilities": {
    "tools": {},
    "resources": {},
    "prompts": {}
  },
  "dashdash": {
    "specVersion": "0.2.0",
    "identity": {
      "name": "acme-mcp",
      "description": "MCP server for Acme project management. Use when user asks to manage projects, track tasks, or collaborate with team members.",
      "emoji": "📊"
    },
    "accessLevel": "interact",
    "alternativeAccess": {
      "cliUrl": "https://acme.com/cli",
      "apiUrl": "https://api.acme.com/docs",
      "webUrl": "https://acme.com/app"
    },
    "install": {
      "npm": "@acme/mcp-server",
      "docker": "acme/mcp-server:latest"
    },
    "requires": {
      "auth": ["api-key"],
      "env": ["ACME_API_KEY"],
      "scopes": ["read:projects", "write:tasks"]
    },
    "invocation": {
      "modelInvocable": true,
      "userInvocable": true
    },
    "rateLimit": "1000/hour",
    "homepage": "https://acme.com",
    "repository": "https://github.com/acme/acme-mcp"
  }
}
```

#### 3.1.2 Dashdash Extension Fields

The `dashdash` extension object contains:

**Required Fields (for Skill Creation):**

- **specVersion** (REQUIRED): Version of this specification. MUST be `0.2.0` or later.

- **identity** (REQUIRED): Server identity for skill creation.
  - **name**: Canonical name. Lowercase letters, numbers, hyphens. Max 64 chars.
  - **description**: What the server does and when to use it. MUST include trigger phrases.
  - **emoji** (OPTIONAL): Display emoji.

- **accessLevel** (REQUIRED): Access control level. Valid values:
  - `read` — Read-only operations
  - `interact` — Can modify data
  - `full` — Unrestricted access

- **alternativeAccess** (REQUIRED): URLs to alternative access methods.
  - **cliUrl**: CLI documentation URL or `null`
  - **apiUrl**: REST API documentation URL or `null`
  - **webUrl**: Web interface URL or `null`

**Recommended Fields:**

- **install** (RECOMMENDED): Installation methods.
  ```json
  {
    "npm": "@org/mcp-server",
    "pip": "mcp-server-name",
    "docker": "org/mcp-server:tag",
    "brew": "org/tap/mcp-server"
  }
  ```

- **requires** (RECOMMENDED): Runtime requirements.
  ```json
  {
    "auth": ["api-key", "oauth2"],
    "env": ["API_KEY", "API_SECRET"],
    "scopes": ["read:data", "write:data"]
  }
  ```

- **invocation** (RECOMMENDED): Skill framework integration.
  ```json
  {
    "modelInvocable": true,
    "userInvocable": true,
    "allowedTools": ["mcp"]
  }
  ```

- **rateLimit** (RECOMMENDED): Human-readable rate limit string.

- **homepage** (RECOMMENDED): Primary documentation URL.

- **repository** (RECOMMENDED): Source code repository URL.

**Optional Fields:**

- **statusPage** (OPTIONAL): Service status page URL.

- **contentVersion** (OPTIONAL): Version of the metadata content.

- **lastUpdated** (OPTIONAL): ISO 8601 date of last update.

### 3.2 Enhanced Tool Metadata

#### 3.2.1 Tool Extensions

MCP servers SHOULD extend tool definitions with additional metadata:

```json
{
  "name": "create_project",
  "description": "Create a new project in Acme",
  "inputSchema": {
    "type": "object",
    "required": ["name"],
    "properties": {
      "name": {
        "type": "string",
        "description": "Project name"
      },
      "type": {
        "type": "string",
        "enum": ["public", "private"],
        "default": "private"
      }
    }
  },
  "dashdash": {
    "category": "projects",
    "operationType": "write",
    "idempotent": false,
    "idempotentWithKey": true,
    "sideEffects": ["creates resource", "sends webhook"],
    "reversible": true,
    "reverseMethod": "delete_project",
    "cliEquivalent": "acme project create --name {name} --type {type}",
    "apiEquivalent": "POST /api/projects",
    "rateLimit": "100/hour",
    "examples": [
      {
        "description": "Create a private project",
        "input": {"name": "my-project", "type": "private"},
        "output": {"id": "proj_123", "name": "my-project"}
      }
    ],
    "errors": [
      {
        "code": "DUPLICATE_NAME",
        "description": "Project with this name already exists",
        "resolution": "Use a different name or update existing project"
      }
    ]
  }
}
```

#### 3.2.2 Tool Extension Fields

The tool-level `dashdash` extension contains:

**Categorization:**

- **category** (RECOMMENDED): Functional category (e.g., "projects", "tasks", "admin").

- **operationType** (RECOMMENDED): One of `read`, `write`, `delete`, `admin`.

**Operational Semantics:**

- **idempotent** (RECOMMENDED): Boolean. True if safe to retry without side effects.

- **idempotentWithKey** (RECOMMENDED): Boolean. True if idempotent when using idempotency key.

- **sideEffects** (RECOMMENDED): Array of side effects (e.g., "sends email", "creates webhook").

- **reversible** (RECOMMENDED): Boolean. True if operation can be undone.

- **reverseMethod** (RECOMMENDED): Name of tool that reverses this operation.

**Cross-Platform:**

- **cliEquivalent** (RECOMMENDED): Equivalent CLI command with parameter placeholders.

- **apiEquivalent** (RECOMMENDED): Equivalent REST API call.

**Documentation:**

- **rateLimit** (OPTIONAL): Tool-specific rate limit if different from server default.

- **examples** (RECOMMENDED): Array of example invocations with expected outputs.

- **errors** (RECOMMENDED): Array of common errors with resolutions.

### 3.3 AI Help Method

#### 3.3.1 Method Definition

MCP servers SHOULD implement an `ai_help` method that returns comprehensive documentation:

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "ai_help",
  "params": {
    "format": "markdown",
    "section": null
  },
  "id": 1
}
```

**Parameters:**
- **format** (OPTIONAL): Output format. Default `markdown`. Also supports `json`.
- **section** (OPTIONAL): Specific section to return. Default `null` returns all.

**Response (markdown format):**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "content": "---\nname: acme-mcp\ndescription: ...\n---\n\n# Acme MCP Server\n\n> MCP server for project management\n\n## When to Use\n\n...",
    "contentType": "text/markdown"
  },
  "id": 1
}
```

**Response (json format):**
```json
{
  "jsonrpc": "2.0",
  "result": {
    "metadata": {
      "name": "acme-mcp",
      "description": "...",
      "specVersion": "0.2.0"
    },
    "sections": {
      "whenToUse": ["Manage projects", "Track tasks"],
      "quickReference": [...],
      "authentication": "...",
      "rateLimits": {...}
    },
    "contentType": "application/json"
  },
  "id": 1
}
```

#### 3.3.2 Markdown Content Requirements

When `format` is `markdown`, the response MUST follow the same structure as the CLI `--ai-help` and web llms.txt specifications:

```markdown
---
# Server Identity
name: acme-mcp
description: >
  MCP server for Acme project management.
  Use when user asks to manage projects, track tasks, or collaborate with team members.

# Specification
spec-url: https://github.com/visionik/dashdash
spec-version: 0.2.0

# Access Level
access-level: interact

# Alternative Access Methods
cli-url: https://acme.com/cli
api-url: https://api.acme.com/docs
web-url: https://acme.com/app

# Installation
install:
  npm: "@acme/mcp-server"
  docker: "acme/mcp-server:latest"

# Requirements
requires:
  auth:
    - api-key
  env:
    - ACME_API_KEY

# Invocation
invocation:
  model-invocable: true
  user-invocable: true
---

# Acme MCP Server

> MCP server for project management and task tracking

## When to Use

Use this MCP server when the user asks to:
- Manage projects or project settings
- Track, create, or update tasks
- Collaborate with team members
- Generate project reports

Do NOT use this MCP server for:
- File storage (use filesystem MCP or dedicated storage)
- Code repositories (use GitHub MCP)
- Personal todo lists (use local tools)

## Quick Reference

**Project Tools:**
- `list_projects` — List all accessible projects
- `get_project` — Get project details by ID
- `create_project` — Create a new project
- `update_project` — Update project settings
- `delete_project` — Delete a project

**Task Tools:**
- `list_tasks` — List tasks in a project
- `create_task` — Create a new task
- `update_task` — Update task details
- `complete_task` — Mark task as complete

## Installation

### npm (RECOMMENDED)

```bash
npm install -g @acme/mcp-server
```

### Docker

```bash
docker run -e ACME_API_KEY=your_key acme/mcp-server:latest
```

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "acme": {
      "command": "npx",
      "args": ["@acme/mcp-server"],
      "env": {
        "ACME_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

## Authentication

### API Key (REQUIRED)

1. Register at https://acme.com/signup
2. Navigate to Settings > API Keys
3. Generate new key with MCP scopes
4. Set environment variable:
   ```bash
   export ACME_API_KEY="your_key_here"
   ```

### Required Scopes

- `read:projects` — List and view projects
- `write:projects` — Create and update projects
- `read:tasks` — List and view tasks
- `write:tasks` — Create and update tasks

## Rate Limits

- **Default:** 1,000 requests/hour
- **Batch operations:** 100 items per request
- **Concurrent connections:** 10

Rate limit errors return:
```json
{
  "error": {
    "code": -32000,
    "message": "Rate limit exceeded",
    "data": {
      "retryAfter": 60
    }
  }
}
```

## Error Handling

### Common Errors

- `-32600` Invalid Request — Malformed JSON-RPC
- `-32601` Method Not Found — Unknown tool/method
- `-32602` Invalid Params — Schema validation failed
- `-32000` Server Error — Check `data.code` for details:
  - `AUTH_FAILED` — Invalid or expired API key
  - `RATE_LIMITED` — Too many requests
  - `NOT_FOUND` — Resource doesn't exist
  - `PERMISSION_DENIED` — Insufficient scopes

## Tool Reference

### list_projects

List all projects accessible to the authenticated user.

**Parameters:**
- `status` (optional, string): Filter by status (`active`, `archived`, `all`)
- `limit` (optional, integer): Max results (default 50, max 100)
- `cursor` (optional, string): Pagination cursor

**Returns:**
```json
{
  "projects": [
    {"id": "proj_123", "name": "My Project", "status": "active"}
  ],
  "nextCursor": "abc123"
}
```

**CLI Equivalent:** `acme projects list --status active`
**API Equivalent:** `GET /api/projects?status=active`

### create_project

Create a new project.

**Parameters:**
- `name` (required, string): Project name (1-100 chars)
- `type` (optional, string): `public` or `private` (default: `private`)
- `description` (optional, string): Project description

**Returns:**
```json
{
  "id": "proj_456",
  "name": "New Project",
  "type": "private",
  "createdAt": "2026-01-21T00:00:00Z"
}
```

**Side Effects:** Creates project, sends webhook if configured

**CLI Equivalent:** `acme projects create --name "New Project" --type private`
**API Equivalent:** `POST /api/projects`

## Alternative Access Methods

### CLI

Full-featured command-line interface:
- **Install:** `brew install acme/tap/acme-cli` or `npm install -g @acme/cli`
- **Docs:** https://acme.com/cli
- **AI Help:** `acme --ai-help`

### REST API

Direct HTTP API access:
- **Base URL:** https://api.acme.com/v2
- **Docs:** https://api.acme.com/docs
- **OpenAPI:** https://api.acme.com/openapi.json

### Web Interface

Browser-based access:
- **URL:** https://acme.com/app
- **Docs:** https://acme.com/docs

## Best Practices

💡 **Use batch operations** when processing multiple items

💡 **Cache project IDs** to avoid repeated lookups

💡 **Handle rate limits gracefully** with exponential backoff

💡 **Prefer MCP over REST API** for AI agent workflows (structured errors, streaming)

## Troubleshooting

### "AUTH_FAILED" Error

**Cause:** Invalid or expired API key

**Solution:**
1. Verify `ACME_API_KEY` environment variable is set
2. Check key hasn't been revoked at https://acme.com/settings/api
3. Ensure key has required scopes

### "RATE_LIMITED" Error

**Cause:** Too many requests

**Solution:**
1. Check `retryAfter` in error response
2. Implement exponential backoff
3. Consider upgrading plan for higher limits
```

### 3.4 Resource Metadata Enhancement

#### 3.4.1 Resource Extensions

MCP servers SHOULD extend resource definitions:

```json
{
  "uri": "acme://projects/proj_123",
  "name": "Project: My Project",
  "mimeType": "application/json",
  "dashdash": {
    "resourceType": "project",
    "refreshable": true,
    "cacheDuration": 300,
    "webEquivalent": "https://acme.com/app/projects/proj_123",
    "apiEquivalent": "GET /api/projects/proj_123"
  }
}
```

#### 3.4.2 Resource Extension Fields

- **resourceType** (RECOMMENDED): Type classification (e.g., "project", "task", "user").

- **refreshable** (RECOMMENDED): Boolean. True if resource content changes.

- **cacheDuration** (RECOMMENDED): Seconds the resource can be cached.

- **webEquivalent** (RECOMMENDED): URL to view this resource in web interface.

- **apiEquivalent** (RECOMMENDED): REST API endpoint for this resource.

## 4. Discovery Integration

### 4.1 MCP Server Registry

MCP servers MAY register with discovery services using the enhanced metadata:

```json
{
  "name": "acme-mcp",
  "description": "MCP server for Acme project management",
  "specVersion": "0.2.0",
  "install": {
    "npm": "@acme/mcp-server"
  },
  "alternativeAccess": {
    "cliUrl": "https://acme.com/cli",
    "apiUrl": "https://api.acme.com/docs",
    "webUrl": "https://acme.com/app"
  },
  "categories": ["productivity", "project-management"],
  "verified": true
}
```

### 4.2 Cross-Reference with Other Formats

MCP servers SHOULD cross-reference with CLI and web documentation:

- CLI tools SHOULD reference MCP servers in `--ai-help` output
- Web llms.txt files SHOULD reference MCP servers
- MCP servers SHOULD reference CLI and web alternatives

## 5. Security Considerations

### 5.1 Information Disclosure

MCP metadata MUST NOT include:
- API keys, tokens, or credentials
- Internal infrastructure details
- Security vulnerabilities

### 5.2 Access Control

The `accessLevel` field is documentation only. MCP servers MUST implement proper authentication and authorization.

### 5.3 Rate Limiting

MCP servers SHOULD implement rate limiting and return structured rate limit information in error responses.

## 6. Implementation Guidance

### 6.1 TypeScript/JavaScript

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";

const server = new Server({
  name: "acme-mcp",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {},
    resources: {}
  }
});

// Add dashdash metadata to server info
server.setRequestHandler("initialize", async (request) => {
  return {
    protocolVersion: "2024-11-05",
    serverInfo: {
      name: "acme-mcp",
      version: "1.0.0"
    },
    capabilities: {
      tools: {},
      resources: {}
    },
    dashdash: {
      specVersion: "0.2.0",
      identity: {
        name: "acme-mcp",
        description: "MCP server for Acme. Use when user asks to manage projects or tasks."
      },
      accessLevel: "interact",
      alternativeAccess: {
        cliUrl: "https://acme.com/cli",
        apiUrl: "https://api.acme.com/docs",
        webUrl: "https://acme.com/app"
      }
    }
  };
});

// Implement ai_help method
server.setRequestHandler("ai_help", async (request) => {
  const format = request.params?.format || "markdown";
  
  if (format === "markdown") {
    return {
      content: getMarkdownHelp(),
      contentType: "text/markdown"
    };
  } else {
    return {
      metadata: getJsonMetadata(),
      sections: getJsonSections(),
      contentType: "application/json"
    };
  }
});
```

### 6.2 Python

```python
from mcp.server import Server
from mcp.types import InitializeResult

server = Server("acme-mcp")

@server.initialize()
async def handle_initialize():
    return InitializeResult(
        protocol_version="2024-11-05",
        server_info={"name": "acme-mcp", "version": "1.0.0"},
        capabilities={"tools": {}, "resources": {}},
        # Dashdash extension
        dashdash={
            "specVersion": "0.2.0",
            "identity": {
                "name": "acme-mcp",
                "description": "MCP server for Acme. Use when user asks to manage projects."
            },
            "accessLevel": "interact",
            "alternativeAccess": {
                "cliUrl": "https://acme.com/cli",
                "apiUrl": "https://api.acme.com/docs",
                "webUrl": "https://acme.com/app"
            }
        }
    )

@server.method("ai_help")
async def handle_ai_help(format: str = "markdown", section: str = None):
    if format == "markdown":
        return {"content": get_markdown_help(), "contentType": "text/markdown"}
    else:
        return {"metadata": get_json_metadata(), "contentType": "application/json"}
```

### 6.3 Testing

Implementers SHOULD:
- Test that `ai_help` returns valid markdown/JSON
- Verify tool extensions are properly included
- Test with actual AI agents
- Confirm SKILL.md can be generated from output

## 7. Evolution and Versioning

### 7.1 Specification Versioning

This specification uses semantic versioning:
- **MAJOR**: Incompatible changes to required fields
- **MINOR**: New optional fields or features
- **PATCH**: Clarifications and examples

### 7.2 Forward Compatibility

MCP clients MUST:
- Ignore unknown fields in `dashdash` extensions
- Treat missing `dashdash` as valid (no enhancements)
- Degrade gracefully for unsupported features

## 8. References

- [Model Context Protocol](https://modelcontextprotocol.io/) — MCP specification
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for RFCs
- [JSON-RPC 2.0](https://www.jsonrpc.org/specification) — Transport protocol
- [dashdash CLI --ai-help](./proposal-cli--ai-help.md) — CLI specification
- [dashdash Web llms.txt](./proposal--web--llms.txt-enhancements.md) — Web specification
- [Agent Skills](https://agentskills.io) — Open standard for AI agent skills
- [Claude Code Skills](https://code.claude.com/docs/en/skills) — Claude Code skill documentation

## 9. Acknowledgments

This specification builds upon:
- [Model Context Protocol](https://modelcontextprotocol.io/) — The foundational MCP specification
- [Anthropic](https://anthropic.com/) — Creators of MCP
- [dashdash project](https://github.com/visionik/dashdash) — AI agent interaction standards
- Analysis of Clawdbot and Claude Code skill formats

## Appendix A: Complete Example

### Enhanced MCP Server Implementation

See the markdown content in §3.3.2 for a complete example of `ai_help` output.

### Tool Definition with Extensions

```json
{
  "tools": [
    {
      "name": "list_projects",
      "description": "List all projects",
      "inputSchema": {
        "type": "object",
        "properties": {
          "status": {"type": "string", "enum": ["active", "archived", "all"]},
          "limit": {"type": "integer", "minimum": 1, "maximum": 100}
        }
      },
      "dashdash": {
        "category": "projects",
        "operationType": "read",
        "idempotent": true,
        "cliEquivalent": "acme projects list --status {status}",
        "apiEquivalent": "GET /api/projects"
      }
    },
    {
      "name": "create_project",
      "description": "Create a new project",
      "inputSchema": {
        "type": "object",
        "required": ["name"],
        "properties": {
          "name": {"type": "string"},
          "type": {"type": "string", "enum": ["public", "private"]}
        }
      },
      "dashdash": {
        "category": "projects",
        "operationType": "write",
        "idempotent": false,
        "idempotentWithKey": true,
        "sideEffects": ["creates resource", "sends webhook"],
        "reversible": true,
        "reverseMethod": "delete_project",
        "cliEquivalent": "acme projects create --name {name} --type {type}",
        "apiEquivalent": "POST /api/projects"
      }
    }
  ]
}
```

## Appendix B: Implementation Checklist

### Required (for Skill Creation)

- [ ] Add `dashdash` extension to initialize response:
  - [ ] `specVersion` — Specification version
  - [ ] `identity.name` — Server name
  - [ ] `identity.description` — Description with trigger phrases
  - [ ] `accessLevel` — Access control level
  - [ ] `alternativeAccess.cliUrl` — CLI URL or null
  - [ ] `alternativeAccess.apiUrl` — API docs URL or null
  - [ ] `alternativeAccess.webUrl` — Web URL or null
- [ ] Implement `ai_help` method returning markdown with:
  - [ ] YAML front matter
  - [ ] When to Use section
  - [ ] Quick Reference section

### Recommended

- [ ] Add `dashdash` extensions to tools:
  - [ ] `category`
  - [ ] `operationType`
  - [ ] `idempotent`
  - [ ] `cliEquivalent`
  - [ ] `apiEquivalent`
  - [ ] `examples`
  - [ ] `errors`
- [ ] Include installation instructions in metadata
- [ ] Include rate limit information
- [ ] Add authentication documentation

### Validation

- [ ] Test `ai_help` output with AI agent
- [ ] Verify tool extensions are included
- [ ] Confirm SKILL.md can be generated
