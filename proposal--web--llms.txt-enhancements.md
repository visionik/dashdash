# Web llms.txt Enhancements Specification

**Version:** 0.2.0  
**Status:** DRAFT PROPOSAL  
**Last Updated:** 2026-01-21

## Important Note

**This is a draft proposal seeking community feedback.** We are **not** attempting to replace or usurp the excellent work done by the team at [llmstxt.org](https://llmstxt.org/). Their simple, elegant approach to LLM-friendly documentation has created a valuable foundation for the community.

This document proposes **optional enhancements** that could extend llms.txt for interactive web applications while maintaining full backward compatibility with the original specification. We welcome critique, suggestions, and alternative approaches. If the community finds these ideas useful, they could be incorporated into the official llms.txt specification, remain as optional extensions, or be discarded entirely.

**We encourage discussion and collaboration** with the llmstxt.org community before any widespread adoption.

**Source**: This proposal is part of the [dashdash project](https://github.com/visionik/dashdash) exploring AI agent interaction standards.

## 1. Introduction

This document specifies enhancements to the `llms.txt` format for websites and web applications that need to provide detailed operational guidance for AI agents. It builds upon the excellent foundation established by [llmstxt.org](https://llmstxt.org/) and incorporates structured metadata capabilities for skill creation, API interaction, and alternative access method discovery.

### 1.1 Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

### 1.2 Design Priorities

The enhanced llms.txt format MUST serve three priorities, in order:

1. **Enable creation of SKILL.md files** — The content MUST provide all information necessary for AI agents or developers to create skill definition files (such as those used by Claude Code, Clawdbot, or other agent frameworks). This includes service identity, triggering context, and operational guidance.

2. **Enable learning of web and API interaction** — The content MUST document how to interact with the website, web application, or web API effectively. This includes authentication flows, URL patterns, form specifications, and API endpoints.

3. **Enable discovery of alternative access methods** — The content MUST document other ways to interact with the underlying product or service, including CLI tools, MCP servers, and other programmatic access methods.

The content MAY provide additional details beyond these priorities, such as troubleshooting guides, performance characteristics, and advanced usage patterns.

## 2. Rationale

### 2.1 Foundation: Why llmstxt.org Got It Right

The team at [llmstxt.org](https://llmstxt.org/) created an elegant solution that addresses a real need:

**Original llms.txt strengths:**
- Simple, predictable file location (`/llms.txt`)
- Human-readable markdown content
- Minimal structure (H1 title, blockquote summary, H2 sections with links)
- Precise format allowing both classical parsing and LLM consumption
- Optional `/llms-full.txt` for complete content in one file
- Clear distinction from robots.txt (guidance vs. permissions)
- Focus on inference-time usage rather than training

**This proposal builds on that foundation** by exploring optional additions for sites with complex operational requirements.

### 2.2 Problem Statement

Traditional web documentation (FAQs, user guides, API docs) is optimized for human navigation with visual hierarchy, tutorials, and conversational tone. AI agents have different needs:

- **Skill creation support** — AIs need structured metadata to create skill definitions
- **Triggering context** — AIs need to know WHEN to use this service, not just HOW
- **Structured endpoint discovery** — Clear mapping of URLs to capabilities
- **Rate limiting and quotas** — Explicit constraints on AI usage
- **Authentication flows** — Step-by-step credential management
- **Data format specifications** — Comprehensive input/output schemas
- **Error code mapping** — Common failure modes and solutions
- **Permission boundaries** — What AIs can and cannot do
- **Alternative access methods** — CLI, API, or MCP alternatives
- **Form field specifications** — Exact expected formats and validation rules

### 2.3 Benefits

- **Skill generation** — Enables automated or assisted creation of SKILL.md files
- **Backward compatible** — Pure markdown files remain valid
- **Standardized discovery** — RFC 8615 `.well-known` registration
- **Progressive enhancement** — YAML front matter adds structure without breaking readability
- **Single file** — No proliferation of AI-specific files
- **Tool ecosystem** — Follows established patterns (Jekyll, Hugo, etc.)
- **Self-contained documentation** — Travels with the service deployment
- **Access control clarity** — Explicit AI permissions and boundaries

## 3. File Location and Discovery

### 3.1 Primary Location (llmstxt.org Standard)

Per the original llmstxt.org specification, sites MUST serve the file at:

```
https://example.com/llms.txt
```

### 3.2 Well-Known URI (RFC 8615 Alternative)

Sites MAY additionally serve the file at the RFC 8615 well-known location:

```
https://example.com/.well-known/llms.txt
```

If both locations are used, they MUST serve identical content.

**Rationale**: The `.well-known` location follows RFC 8615 for standardized site-wide metadata discovery, while maintaining compatibility with the original llmstxt.org convention.

### 3.3 Full Content Variant

Per llmstxt.org, sites MAY provide a comprehensive version at:

```
https://example.com/llms-full.txt
```

This file contains all documentation inline rather than links to external resources.

### 3.4 Content Type

Sites SHOULD serve the file with:

```http
Content-Type: text/markdown; charset=utf-8
```

### 3.5 Discoverability

#### 3.5.1 robots.txt Integration

Sites SHOULD reference the file in `robots.txt`:

```
# AI Agent Help
User-agent: *
Allow: /llms.txt
Allow: /llms-full.txt

# AI-specific user agents
User-agent: GPTBot
User-agent: ClaudeBot
User-agent: ChatGPT-User
Crawl-delay: 1
```

#### 3.5.2 HTML Meta Tag

Sites MAY include a meta tag:

```html
<meta name="llms-txt" content="/llms.txt">
```

#### 3.5.3 HTTP Header

Sites MAY include a custom header on responses:

```http
X-LLMs-Txt: /llms.txt
```

#### 3.5.4 Link Relation

Sites MAY use a Link header or HTML `<link>` element:

```http
Link: </llms.txt>; rel="llms-help"
```

```html
<link rel="llms-help" href="/llms.txt">
```

## 4. File Format

### 4.1 Base Format (llmstxt.org Standard)

Files MUST be valid CommonMark markdown with:

1. **Required**: Single H1 heading (site/service name)
2. **Required**: Blockquote with brief summary
3. **Required**: One or more H2 sections with content or links
4. **Optional**: YAML front matter (see §4.2)

**Base llms.txt example** (per https://llmstxt.org/):

```markdown
# Example Corp

> Example Corp provides project management tools for teams

## Quick Links

- [API Documentation](https://example.com/api/docs)
- [Getting Started Guide](https://example.com/guide)
- [Support](https://example.com/support)
```

### 4.2 Extended Format with Front Matter

Files MAY include YAML front matter for machine-readable metadata. The front matter is separated from content by `---` markers:

```yaml
---
# Service Identity (REQUIRED for skill creation)
name: example-corp
description: >
  Project management platform for teams.
  Use when user asks to track tasks, manage projects, or collaborate with team members.

# Specification Reference (REQUIRED when using front matter)
spec-url: https://github.com/visionik/dashdash
spec-version: 0.2.0

# Access Control (REQUIRED when using front matter)
access-level: interact

# Alternative Access Methods (REQUIRED when using front matter)
cli-url: https://example.com/cli
mcp-url: none
api-url: https://api.example.com/docs
---

# Example Corp

> Example Corp provides project management tools for teams

...
```

**Rationale**: Follows Jekyll/Hugo/Astro convention where YAML front matter is separated from markdown content. Parsers that don't support YAML will simply display raw front matter as text, maintaining readability.

### 4.3 Front Matter Fields

#### 4.3.1 Required Fields (for Skill Creation)

When using front matter, the following fields MUST be included:

**Service Identity Fields:**

- **name** (REQUIRED): Canonical name for the service. MUST use lowercase letters, numbers, and hyphens only. Maximum 64 characters. This field is REQUIRED for skill creation.

- **description** (REQUIRED): What the service does and when to use it. MUST include trigger phrases such as "Use when user asks to..." or "Use for...". This field is REQUIRED for skill creation as it determines when AI agents should invoke the tool or service.

**Specification Fields:**

- **spec-url** (REQUIRED): URL to the dashdash specification repository.

- **spec-version** (REQUIRED): Version of this specification the file implements. MUST use semantic versioning (e.g., `0.2.0`).

**Access Control Fields:**

- **access-level** (REQUIRED): Access control level. Valid values:
  - `none` — AI agents MUST NOT interact with this site
  - `read` — AI agents MAY read public information only
  - `browse` — AI agents MAY navigate and search but not submit forms
  - `interact` — AI agents MAY submit forms and perform actions (with constraints in content)
  - `full` — AI agents have same access as authenticated users (subject to rate limits)

**Alternative Access Method Fields:**

- **cli-url** (REQUIRED): URL to CLI tool documentation or download page. Use `none` if no CLI exists.

- **mcp-url** (REQUIRED): URL to MCP (Model Context Protocol) server documentation or repository. Use `none` if no MCP exists.

- **api-url** (REQUIRED): URL to REST API documentation. Use `none` if no API exists.

#### 4.3.2 Recommended Fields

The following fields SHOULD be included when applicable:

**API Configuration:**

- **api-endpoint** (RECOMMENDED): Base URL for API requests.

- **rate-limit** (RECOMMENDED): Human-readable rate limit (e.g., "1000/hour", "50/minute").

- **auth-methods** (RECOMMENDED): Supported authentication methods. Valid values: `bearer`, `api-key`, `oauth2`, `cookie`, `basic`, `session`.

**Installation Information:**

- **install** (RECOMMENDED): Installation instructions for CLI tools. Each key represents a package manager, and the value is the package identifier.
  ```yaml
  install:
    brew: "org/tap/formula"
    npm: "@org/package"
    pip: "package-name"
    cargo: "crate-name"
    go: "github.com/org/repo"
  ```

**Requirements:**

- **requires** (RECOMMENDED): Access requirements for the service.
  ```yaml
  requires:
    auth:
      - api-key
      - oauth2
    scopes:
      - read:projects
      - write:tasks
    bins:
      - curl
    os:
      - darwin
      - linux
  ```

**Invocation Control:**

- **invocation** (RECOMMENDED): Controls for AI agent invocation behavior. These fields align with skill framework conventions:
  ```yaml
  invocation:
    model-invocable: true
    user-invocable: true
    allowed-tools:
      - Browser
      - Fetch
  ```
  - **model-invocable**: Boolean. If `true`, AI agents MAY automatically invoke this service. If `false`, the service SHOULD only be used by explicit user request.
  - **user-invocable**: Boolean. If `true`, the service appears in user-facing menus (e.g., `/skill` commands). If `false`, the service is background knowledge only.
  - **allowed-tools**: List of tools/permissions the skill may use when active.

**Documentation Links:**

- **homepage** (RECOMMENDED): URL to the service's primary marketing or landing page.

- **repository** (RECOMMENDED): URL to the source code repository if open source.

- **status-page** (RECOMMENDED): URL to service status page.

#### 4.3.3 Optional Fields

The following fields MAY be included:

- **content-version** (OPTIONAL): Version of the content itself, independent of the specification version.

- **last-updated** (OPTIONAL): ISO 8601 date when the content was last updated.

- **emoji** (OPTIONAL): Single emoji character representing the service for display purposes.

- **cors-enabled** (OPTIONAL): Boolean indicating whether CORS is supported.

- **websocket-endpoint** (OPTIONAL): WebSocket endpoint URL if available.

- **subdirectory-help** (OPTIONAL): Boolean indicating whether subdirectory llms.txt files exist (see §6).

### 4.4 Markdown Content Style

Following the front matter, content MUST be markdown with:
- Clear section headers (using `##` and `###`)
- Code blocks with appropriate language tags
- Lists for enumeration
- Emphasis for warnings (e.g., ⚠️, ❌, ✅ emoji)

Content SHOULD:
- Be verbose and explicit rather than concise
- Use active voice and imperative mood
- Include complete, executable examples
- Repeat important information when necessary

## 5. Content Requirements

### 5.1 Required from llmstxt.org Standard

Per https://llmstxt.org/, content MUST include:

1. **H1 title** — Site/service name
2. **Blockquote summary** — Brief description (1-2 sentences)
3. **H2 sections** — At least one section with relevant content or links

### 5.2 Required Sections for Skill Creation

When targeting skill creation compatibility, content MUST include:

#### 5.2.1 When to Use Section

The content MUST include a "When to Use" section near the top of the document. This section is REQUIRED for skill creation and MUST clearly specify the contexts in which an AI agent should use this service.

The section MUST:
- List specific user intents or queries that should trigger this service
- Use action-oriented language ("Use when user asks to...")
- Be extractable for use in skill descriptions
- Include negative examples where appropriate

Example:
```markdown
## When to Use

Use this service when the user asks to:
- Track or manage project tasks
- View project status or progress
- Assign tasks to team members
- Generate project reports
- Set up project notifications or webhooks

Do NOT use this service for:
- Personal todo lists (use local todo apps instead)
- Non-team/individual task management
- File storage (use dedicated storage services)
- Code repositories (use GitHub, GitLab, etc.)
```

#### 5.2.2 Quick Reference Section

The content MUST include a "Quick Reference" section that provides a concise summary of common operations. This section SHOULD:
- Include 10-20 of the most common operations
- Use consistent formatting
- Be suitable for inclusion in a skill body

Example:
```markdown
## Quick Reference

**API Operations:**
- **List projects:** `GET /api/projects`
- **Get project:** `GET /api/projects/{id}`
- **Create project:** `POST /api/projects`
- **Update project:** `PATCH /api/projects/{id}`
- **Delete project:** `DELETE /api/projects/{id}`
- **List tasks:** `GET /api/projects/{id}/tasks`
- **Create task:** `POST /api/projects/{id}/tasks`
- **Search:** `GET /api/search?q={query}`

**Web Navigation:**
- **Dashboard:** `/app/dashboard`
- **Project list:** `/app/projects`
- **Task board:** `/app/projects/{id}/board`
- **Settings:** `/app/settings`
```

### 5.3 Recommended Sections for Interactive Sites

For sites with APIs, forms, or programmatic access, content SHOULD include:

#### 5.3.1 Core Sections

- **Overview** — Service purpose and primary capabilities
- **Authentication** (§5.4) — How to obtain and use credentials
- **Rate Limits and Performance** (§5.8) — Request quotas and headers
- **Error Handling** — Common HTTP status codes and responses
- **Quick Start** — Minimal example to verify access

#### 5.3.2 Technical Sections

- **URL Patterns** — Key endpoint patterns and routing
- **Data Formats** — Supported input/output formats (JSON, form-data, etc.)
- **Date/Time Handling** (§5.5) — Supported formats (ISO 8601 RECOMMENDED)
- **Form Specifications** (§5.6) — Field names, validation rules
- **Input Schema** (§5.9) — Request body and parameter schemas
- **Output Schema** (§5.10) — Response structures and guarantees

#### 5.3.3 Operational Sections

- **Allowed Operations** — What AI agents MAY do
- **Forbidden Operations** — What AI agents MUST NOT do
- **Operational Semantics** (§5.11) — Idempotency, side effects, pagination
- **Webhooks** — Async notification mechanisms (if applicable)

#### 5.3.4 Discovery Sections

- **Alternative Access Methods** — Links to CLI, API, MCP
- **Troubleshooting** — Common errors with solutions
- **Best Practices** — AI-specific recommendations

### 5.4 Authentication Section

Documentation MUST detail authentication mechanisms when applicable:

```markdown
## Authentication

### API Key (RECOMMENDED for AI agents)

**Obtain key:**
1. Register at https://example.com/register
2. Navigate to Settings > API Keys
3. Generate new key with desired scopes

**Use API Key:**
```http
GET /api/projects HTTP/1.1
Host: api.example.com
Authorization: Bearer YOUR_API_KEY_HERE
Accept: application/json
```

### OAuth 2.0 (for delegated access)

**Not recommended for AI agents** — requires interactive browser flow

### Required Scopes

- `read:projects` — List and view projects
- `write:projects` — Create and update projects
- `delete:projects` — Delete projects (admin only)
```

### 5.5 Date/Time Handling

If the site accepts date or time parameters, the documentation MUST:
- List ALL supported formats with examples
- List common UNSUPPORTED formats with explanations
- Include timezone handling guidance

Example:
```markdown
## Date/Time Formats

### ✅ SUPPORTED FORMATS

- `2026-01-21` — ISO 8601 date
- `2026-01-21T15:30:00Z` — ISO 8601 datetime (UTC)
- `2026-01-21T10:30:00-05:00` — ISO 8601 with timezone offset

### ❌ UNSUPPORTED FORMATS

- `01/21/2026` — Ambiguous MM/DD/YYYY
  **Error:** "Invalid date format"
  **Fix:** Use ISO 8601: `2026-01-21`

- `"7 days ago"` — Natural language not supported
  **Error:** "Invalid date specification"
  **Fix:** Calculate absolute date
```

### 5.6 Form Specifications

For sites with forms, documentation MUST include:
- Field names (exact HTML `name` attributes or JSON keys)
- Expected data types and formats
- Validation rules (min/max length, patterns, etc.)
- Required vs. optional fields
- Default values if any

Example:
```markdown
## Form Specifications

### Create Project

**Endpoint:** `POST /api/projects`
**Content-Type:** `application/json`

**Fields:**
- `name` (REQUIRED, string): Project name, 1-100 chars, alphanumeric with hyphens
- `type` (REQUIRED, string): One of `public`, `private`, `internal`
- `description` (OPTIONAL, string): Max 1000 chars
- `tags` (OPTIONAL, array): Up to 10 string tags

**Example:**
```json
{
  "name": "my-project",
  "type": "private",
  "description": "A sample project",
  "tags": ["backend", "api"]
}
```
```

### 5.7 Error Handling

Documentation SHOULD include:
- Common HTTP status codes and meanings
- Error response format
- Troubleshooting guidance

Example:
```markdown
## Error Handling

### HTTP Status Codes

- `200 OK` — Success
- `201 Created` — Resource created
- `400 Bad Request` — Invalid request format
- `401 Unauthorized` — Missing/invalid credentials
- `403 Forbidden` — Valid auth but insufficient permissions
- `404 Not Found` — Resource doesn't exist
- `429 Too Many Requests` — Rate limit exceeded
- `500 Internal Server Error` — Retry with exponential backoff

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Field 'name' is required",
    "details": {
      "field": "name",
      "constraint": "required"
    }
  }
}
```
```

### 5.8 Rate Limits and Performance

Documentation MUST specify rate limits and performance characteristics:

```markdown
## Rate Limits

### Limits by Tier

- **Free:** 100 requests/hour, 10/min burst
- **Pro:** 1,000 requests/hour, 100/min burst
- **Enterprise:** Custom limits

### Rate Limit Headers

All responses include:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1736265600
Retry-After: 60
```

### When Rate Limited (429)

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retry_after": 60
  }
}
```

💡 **Best Practice:** Check `X-RateLimit-Remaining` proactively and back off before hitting limits.

### Timeouts

- Connection: 10 seconds
- Request processing: 30 seconds
- Long-running operations: Use async pattern with job polling

### Payload Limits

- Request body: 10 MB
- File uploads: 100 MB
- Batch operations: 1,000 items
```

### 5.9 Input Schema

For APIs with structured inputs, documentation SHOULD include JSON Schema or equivalent:

```markdown
## Input Schema

### POST /api/projects

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "type"],
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100,
      "pattern": "^[a-zA-Z0-9-]+$"
    },
    "type": {
      "type": "string",
      "enum": ["public", "private", "internal"]
    },
    "tags": {
      "type": "array",
      "items": {"type": "string"},
      "maxItems": 10
    }
  }
}
```

**Constraints:**
- `type: private` requires authenticated user
- Cannot specify both `--template` and `--from-scratch`
```

### 5.10 Output Schema

For APIs, documentation SHOULD specify response structures:

```markdown
## Output Schema

### GET /api/projects

**Success (200 OK):**
```json
{
  "data": [
    {
      "id": "string (proj_[a-z0-9]+)",
      "name": "string",
      "type": "string (public|private|internal)",
      "status": "string (active|archived)",
      "created_at": "string (ISO 8601)",
      "updated_at": "string (ISO 8601, nullable)"
    }
  ],
  "pagination": {
    "total": "integer",
    "page": "integer",
    "per_page": "integer",
    "has_next": "boolean",
    "next_cursor": "string (nullable)"
  }
}
```

**Field Guarantees:**
- `id`, `name`, `type`, `status`, `created_at` always present
- `updated_at` null if never modified
- `next_cursor` null on last page
```

### 5.11 Operational Semantics

Documentation SHOULD specify operational behavior:

```markdown
## Operational Semantics

### Idempotency

**Idempotent Methods:**
- `GET`, `HEAD`, `OPTIONS` — Safe, no side effects
- `PUT` — Replaces entire resource, repeatable
- `DELETE` — Returns success even if already deleted

**Idempotent with Key:**
- `POST` with `Idempotency-Key` header — Safe to retry within 24h

**Non-Idempotent:**
- `POST` without idempotency key — May create duplicates
- `PATCH` on counters — Each call modifies state

### Side Effects

**Read Operations (No Side Effects):**
- `GET /api/projects`

**Write Operations (Reversible):**
- `POST /api/projects` — Creates resource (manual DELETE to reverse)
- `PATCH /api/projects/{id}` — Updates resource (previous version in history)

**Destructive Operations (Time-limited):**
- `DELETE /api/projects/{id}` — 30-day soft delete, can restore

### Pagination

**Cursor-based (RECOMMENDED):**
```http
GET /api/projects?limit=50&cursor=abc123
```

💡 Cursor pagination is stable across concurrent modifications.

### Partial Failures

Batch operations return per-item status:
```json
{
  "results": [
    {"id": "id1", "status": "success"},
    {"id": "id2", "status": "error", "error": "Not found"}
  ],
  "summary": {"succeeded": 1, "failed": 1}
}
```
```

## 6. Hierarchical Structure (Optional)

### 6.1 Subdirectory Files

Large sites MAY provide additional llms.txt files in subdirectories:

```
https://example.com/llms.txt              # Root - general info
https://example.com/api/llms.txt          # API-specific
https://example.com/app/llms.txt          # Application-specific
https://example.com/admin/llms.txt        # Admin-specific
```

### 6.2 Root File Requirements

If hierarchical files exist, the root file:
- MUST set `subdirectory-help: true` in front matter
- SHOULD include a "Site Structure" section listing subdirectory files
- MUST define global constraints (rate limits, authentication)

### 6.3 Inheritance

Subdirectory files:
- Inherit front matter from parent unless overridden
- May have more restrictive `access-level`
- Should reference the root file for general context

## 7. Security Considerations

### 7.1 Information Disclosure

AI help content MUST NOT include:
- Real API keys, tokens, or credentials (use placeholders like `YOUR_API_KEY_HERE`)
- Internal infrastructure details
- Private endpoint URLs
- Security vulnerabilities or known exploits
- Overly precise rate limits that enable abuse

### 7.2 Access Control Enforcement

The `access-level` field is documentation only. Sites MUST implement technical access controls:
- Rate limiting per API key/IP
- Authentication enforcement
- Permission checking
- Audit logging

### 7.3 User Agent Identification

Sites SHOULD require AI agents to:
- Use descriptive User-Agent headers
- Identify themselves as automated agents
- Respect rate limits and `Retry-After` headers

```markdown
## User-Agent Requirements

AI agents MUST identify themselves:

✅ GOOD:
```http
User-Agent: MyAIBot/1.0 (+https://example.com/bot-info)
```

❌ BAD (pretending to be browser):
```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```
```

## 8. Implementation Guidance

### 8.1 Static vs. Dynamic

Implementations MAY:
1. Serve static markdown files (RECOMMENDED)
2. Generate content dynamically based on user context

Static files are preferred for cacheability and transparency.

### 8.2 Testing

Implementers SHOULD:
- Test with actual AI agents (ChatGPT, Claude, etc.)
- Validate YAML front matter parses correctly
- Verify all examples are executable
- Monitor for AI-related errors in logs
- Update content when APIs/structure changes
- Confirm a valid SKILL.md can be generated from the content

### 8.3 Platform Examples

#### Nginx

```nginx
location = /llms.txt {
    root /var/www/html;
    add_header Content-Type "text/markdown; charset=utf-8";
}

location = /llms-full.txt {
    root /var/www/html;
    add_header Content-Type "text/markdown; charset=utf-8";
}
```

#### Express.js

```javascript
app.get('/llms.txt', (req, res) => {
  res.type('text/markdown');
  res.sendFile(path.join(__dirname, 'public', 'llms.txt'));
});

app.get('/llms-full.txt', (req, res) => {
  res.type('text/markdown');
  res.sendFile(path.join(__dirname, 'public', 'llms-full.txt'));
});
```

#### Next.js

```typescript
// app/llms.txt/route.ts
import fs from 'fs';
import path from 'path';

export async function GET() {
  const content = fs.readFileSync(
    path.join(process.cwd(), 'public', 'llms.txt'),
    'utf-8'
  );
  
  return new Response(content, {
    headers: {
      'Content-Type': 'text/markdown; charset=utf-8',
    },
  });
}
```

## 9. Evolution and Versioning

### 9.1 Specification Versioning

This specification uses semantic versioning:
- **MAJOR**: Incompatible changes to required fields
- **MINOR**: New optional fields or recommended sections
- **PATCH**: Clarifications and examples

### 9.2 Content Versioning

Individual sites SHOULD version their content using the `content-version` front matter field.

### 9.3 Forward Compatibility

Parsers MUST:
- Ignore unknown front matter fields
- Treat missing front matter as valid (pure markdown per llmstxt.org)
- Degrade gracefully for unsupported features

## 10. References

- [llms.txt](https://llmstxt.org/) — Original llms.txt specification
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615.html) — Well-Known URIs
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) — Key words for RFCs
- [CommonMark](https://commonmark.org/) — Markdown specification
- [YAML 1.2](https://yaml.org/spec/1.2.2/) — YAML specification
- [dashdash CLI --ai-help](./proposal-cli--ai-help.md) — CLI specification from same project
- [Agent Skills](https://agentskills.io) — Open standard for AI agent skills
- [Claude Code Skills](https://code.claude.com/docs/en/skills) — Claude Code skill documentation
- [Model Context Protocol](https://modelcontextprotocol.io/) — MCP specification

## 11. Acknowledgments

We are deeply grateful to the team behind [llmstxt.org](https://llmstxt.org/) for their pioneering work in creating a simple, elegant standard for LLM-friendly documentation. Their insight that "markdown is human and LLM readable, but is also in a precise format allowing fixed processing methods" has provided a crucial foundation for making web content more accessible to AI agents.

The original llms.txt specification's focus on simplicity and human readability is exemplary, and this proposal attempts to honor that spirit while exploring additional capabilities for complex interactive applications.

This document also builds upon:
- [llmstxt.org](https://llmstxt.org/) — The foundational llms.txt convention
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615.html) — IETF Well-Known URIs standard
- [dashdash project](https://github.com/visionik/dashdash) — AI agent interaction standards
- Analysis of Clawdbot and Claude Code skill formats
- Real-world AI agent interaction patterns

**We invite the llmstxt.org team and the broader community to provide feedback, critique, and alternative approaches.** This proposal is meant to start a conversation, not to dictate a solution.

## Appendix A: Complete Example

### llms.txt with Full Enhancements

```markdown
---
# Service Identity
name: acme-pm
description: >
  Project management platform for teams.
  Use when user asks to track tasks, manage projects, collaborate with team members,
  or generate project reports.

# Specification
spec-url: https://github.com/visionik/dashdash
spec-version: 0.2.0

# Access Control
access-level: interact

# Alternative Access Methods
cli-url: https://acme.com/cli
mcp-url: none
api-url: https://api.acme.com/docs

# API Configuration
api-endpoint: https://api.acme.com/v2
rate-limit: 1000/hour
auth-methods:
  - bearer
  - api-key

# Installation
install:
  brew: acme/tap/acme-cli
  npm: "@acme/cli"

# Requirements
requires:
  auth:
    - api-key
  scopes:
    - read:projects
    - write:tasks

# Invocation
invocation:
  model-invocable: true
  user-invocable: true

# Documentation
homepage: https://acme.com
status-page: https://status.acme.com

# Versioning
content-version: 2.1.0
last-updated: 2026-01-21
---

# Acme Project Management

> Team project management and collaboration platform

## When to Use

Use this service when the user asks to:
- Manage projects or project settings
- Track, create, or update tasks
- Collaborate with team members
- Generate project reports or exports
- Set up webhooks or integrations

Do NOT use this service for:
- Personal todo lists (use local todo apps instead)
- File storage (use dedicated storage services)
- Code repositories (use GitHub, GitLab, etc.)

## Quick Reference

**API Operations:**
- **List projects:** `GET /api/projects`
- **Get project:** `GET /api/projects/{id}`
- **Create project:** `POST /api/projects`
- **Update project:** `PATCH /api/projects/{id}`
- **Delete project:** `DELETE /api/projects/{id}`
- **List tasks:** `GET /api/projects/{id}/tasks`
- **Create task:** `POST /api/projects/{id}/tasks`
- **Search:** `GET /api/search?q={query}`
- **Export:** `POST /api/export`

**Web Navigation:**
- **Dashboard:** `/app/dashboard`
- **Project list:** `/app/projects`
- **Task board:** `/app/projects/{id}/board`
- **Settings:** `/app/settings`
- **API Keys:** `/app/settings/api`

## Documentation Links

- [API Reference](https://api.acme.com/docs)
- [Getting Started](https://acme.com/docs/quickstart)
- [Authentication Guide](https://acme.com/docs/auth)
- [Webhooks](https://acme.com/docs/webhooks)

## Authentication

### API Key (RECOMMENDED)

1. Register at https://acme.com/signup
2. Navigate to Settings > API Keys
3. Generate new key

```http
GET /api/projects HTTP/1.1
Host: api.acme.com
Authorization: Bearer YOUR_API_KEY_HERE
Accept: application/json
```

## Rate Limits

- **Free:** 100 requests/hour
- **Pro:** 1,000 requests/hour
- **Enterprise:** Custom

Headers:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1736265600
```

## Allowed Operations

✅ AI agents MAY:
- Read project data
- Create and update tasks
- Search and filter
- Export data (JSON/CSV)

## Forbidden Operations

❌ AI agents MUST NOT:
- Delete projects (requires human confirmation)
- Modify user permissions
- Send notifications to users
- Exceed rate limits

## Error Handling

- `200 OK` — Success
- `400 Bad Request` — Invalid request
- `401 Unauthorized` — Missing/invalid credentials
- `403 Forbidden` — Insufficient permissions
- `404 Not Found` — Resource doesn't exist
- `429 Too Many Requests` — Rate limited

## Quick Start

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Accept: application/json" \
     https://api.acme.com/v2/projects
```

## Alternative Access

- **CLI:** https://acme.com/cli (`brew install acme/tap/acme-cli`)
- **MCP:** Not available
- **API Docs:** https://api.acme.com/docs
- **Status:** https://status.acme.com
```

## Appendix B: Implementation Checklist

### Required (for Skill Creation)

- [ ] Serve llms.txt at `/llms.txt`
- [ ] Include required front matter fields (when using front matter):
  - [ ] `name` — Service name for skill creation
  - [ ] `description` — What it does and when to use it
  - [ ] `spec-url` — Link to dashdash repo
  - [ ] `spec-version` — Specification version
  - [ ] `access-level` — Access control level
  - [ ] `cli-url` — CLI URL or `none`
  - [ ] `mcp-url` — MCP URL or `none`
  - [ ] `api-url` — API docs URL or `none`
- [ ] Include required content:
  - [ ] H1 title (per llmstxt.org)
  - [ ] Blockquote summary (per llmstxt.org)
  - [ ] When to Use section
  - [ ] Quick Reference section

### Recommended

- [ ] Include recommended front matter:
  - [ ] `api-endpoint`
  - [ ] `rate-limit`
  - [ ] `auth-methods`
  - [ ] `install`
  - [ ] `requires`
  - [ ] `invocation`
  - [ ] `homepage`
  - [ ] `status-page`
- [ ] Provide llms-full.txt with inline content
- [ ] Reference in robots.txt
- [ ] Include recommended sections:
  - [ ] Authentication
  - [ ] Rate Limits
  - [ ] Error Handling
  - [ ] Allowed/Forbidden Operations
  - [ ] Quick Start
  - [ ] Alternative Access Methods

### Validation

- [ ] Test with actual AI agent
- [ ] Validate YAML front matter
- [ ] Verify examples are executable
- [ ] Confirm SKILL.md can be generated

## Appendix C: Comparison with Other Formats

### vs. Original llms.txt

- **Fully backward compatible** — Pure llmstxt.org files remain valid
- **Adds optional YAML front matter** — For structured metadata
- **Adds required sections** — "When to Use", "Quick Reference" for skill creation
- **Adds discovery fields** — CLI, MCP, API URLs

### vs. OpenAPI/Swagger

- **llms.txt:** Human-readable guidance including non-API interactions
- **OpenAPI:** Machine-readable API specification
- **Relationship:** llms.txt can link to OpenAPI specs

### vs. robots.txt

- **robots.txt:** Crawl permissions
- **llms.txt:** How to interact
- **Relationship:** Complementary, both should be used
