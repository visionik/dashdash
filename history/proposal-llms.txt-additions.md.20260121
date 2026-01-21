# llms.txt Proposed Additions

**Version:** 1.0.0  
**Status:** DRAFT PROPOSAL  
**Last Updated:** 2026-01-12

## Important Note

**This is a draft proposal seeking community feedback.** We are **not** attempting to replace or usurp the excellent work done by the team at [llmstxt.org](https://llmstxt.org/). Their simple, elegant approach to LLM-friendly documentation has created a valuable foundation for the community.

This document proposes **optional additions** that could enhance llms.txt for interactive web applications while maintaining full backward compatibility with the original specification. We welcome critique, suggestions, and alternative approaches. If the community finds these ideas useful, they could be incorporated into the official llms.txt specification, remain as optional extensions, or be discarded entirely.

**We encourage discussion and collaboration** with the llms.txt community before any widespread adoption.

**Source**: This proposal is part of the [dashdash project](https://github.com/visionik/dashdash) exploring AI agent interaction standards.

## 1. Introduction

This document proposes extensions to the `llms.txt` format for websites that need to provide detailed operational guidance for AI agents. It builds upon the excellent foundation established by [llmstxt.org](https://llmstxt.org/) and incorporates the standardization framework from [RFC 8615 Well-Known URIs](https://www.rfc-editor.org/rfc/rfc8615.html), along with structured metadata capabilities inspired by the dashdash project.

### 1.1 Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

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

### 2.2 Proposed Additions

This document proposes the following **optional** enhancements:
- RFC 8615-compliant `.well-known` URI registration (for standardization)
- Optional YAML front matter for machine-readable metadata
- Structured guidance for interactive web applications (APIs, forms, authentication)
- Access control levels, rate limits, and operational semantics

**Important:** These additions are designed to be **completely optional and backward compatible**. A pure llms.txt file as defined by llmstxt.org remains 100% valid under this proposal.

### 2.3 Benefits

- **Backward compatible**: Pure markdown files remain valid
- **Standardized discovery**: RFC 8615 `.well-known` registration
- **Progressive enhancement**: YAML front matter adds structure without breaking readability
- **Single file**: No proliferation of AI-specific files
- **Tool ecosystem**: Follows established patterns (Jekyll, Hugo, etc.)

## 3. File Location and Discovery

### 3.1 Well-Known URI

Sites MUST register and serve the file at:

```
https://example.com/.well-known/llms.txt
```

**Rationale**: Follows RFC 8615 for standardized site-wide metadata discovery.

### 3.2 Root Redirect

Sites SHOULD provide a redirect from `/llms.txt` to `/.well-known/llms.txt`:

```http
GET /llms.txt HTTP/1.1
Host: example.com

HTTP/1.1 301 Moved Permanently
Location: /.well-known/llms.txt
```

**Rationale**: Maintains compatibility with existing llms.txt convention while standardizing on `.well-known` location.

### 3.3 IANA Registration

The URI suffix `llms.txt` SHOULD be registered with IANA per RFC 8615 Section 3.1:

- **URI suffix**: `llms.txt`
- **Change controller**: Community specification
- **Specification document**: This document
- **Status**: Provisional (pending implementation adoption)
- **Related information**: https://llmstxt.org/

### 3.4 Content Negotiation

Sites MAY support content negotiation:

```http
GET /.well-known/llms.txt HTTP/1.1
Accept: text/markdown

HTTP/1.1 200 OK
Content-Type: text/markdown
```

```http
GET /.well-known/llms.txt HTTP/1.1
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json
{
  "metadata": {...},
  "content": "..."
}
```

## 4. File Format

### 4.1 Base Format

Files MUST be valid CommonMark markdown with:

1. **Required**: Single H1 heading (site/service name)
2. **Required**: Blockquote with brief summary
3. **Required**: One or more H2 sections
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

Files MAY include YAML front matter for machine-readable metadata:

```yaml
---
# Required fields
acl: interact
version: 1.0.0

# Optional fields  
api_endpoint: https://api.example.com/v2
rate_limit: 1000/hour
auth_methods: [bearer, api_key]
cli_available: true
mcp_available: false

# Optional links
api_docs: https://api.example.com/docs
cli_docs: https://example.com/cli
status_page: https://status.example.com
---

# Example Corp

> Example Corp provides project management tools for teams

## Authentication

...
```

**Rationale**: Follows Jekyll/Hugo/Astro convention where YAML front matter is separated from markdown content. Parsers that don't support YAML will simply display raw front matter as text, maintaining readability.

### 4.3 Front Matter Fields

#### 4.3.1 Required Fields

- **acl** (string): Access control level (see §4.4)
- **version** (string): Content version (semantic versioning recommended)

#### 4.3.2 Recommended Fields

- **api_endpoint** (string): Base URL for API
- **rate_limit** (string): Human-readable rate limit (e.g., "1000/hour", "50/minute")
- **auth_methods** (array): Supported authentication methods (e.g., `bearer`, `api_key`, `oauth2`, `cookie`)

#### 4.3.3 Optional Fields

- **cli_available** (boolean): CLI tool exists
- **mcp_available** (boolean): MCP server exists
- **api_docs** (URL): Link to API documentation
- **cli_docs** (URL): Link to CLI documentation
- **status_page** (URL): Service status page
- **cors_enabled** (boolean): CORS support for client-side requests
- **websocket_endpoint** (URL): WebSocket endpoint if available
- **last_updated** (ISO 8601 date): Content last modified date

### 4.4 Access Control Levels

The `acl` field MUST be one of:

- **none** - AI agents MUST NOT interact with this site
- **read** - AI agents MAY read public information only
- **browse** - AI agents MAY navigate and search but not submit forms
- **interact** - AI agents MAY submit forms and perform actions (with constraints specified in content)
- **full** - AI agents have same access as authenticated users (subject to rate limits)

Sites SHOULD default to `read` unless explicitly designed for AI interaction.

## 5. Content Requirements

### 5.1 Required from llms.txt Standard

Per https://llmstxt.org/, content MUST include:

1. **H1 title** - Site/service name
2. **Blockquote summary** - Brief description (1-2 sentences)
3. **H2 sections** - At least one section with relevant content

### 5.2 Recommended Sections for Interactive Sites

For sites with APIs, forms, or programmatic access, content SHOULD include:

#### 5.2.1 Core Sections

- **Authentication** - How to obtain and use credentials
- **Rate Limits** - Request quotas and headers
- **Error Handling** - Common HTTP status codes and responses
- **Quick Start** - Minimal example to verify access

#### 5.2.2 Technical Sections

- **URL Patterns** - Key endpoint patterns
- **Data Formats** - Supported input/output formats (JSON, form-data, etc.)
- **Date/Time Handling** - Supported formats (ISO 8601 recommended)
- **Pagination** - How to traverse large result sets

#### 5.2.3 Operational Sections

- **Allowed Operations** - What AI agents may do
- **Forbidden Operations** - What AI agents must not do
- **Idempotency** - Safe retry behavior
- **Webhooks** - Async notification mechanisms (if applicable)

### 5.3 Content Style

Content SHOULD:
- Be verbose and explicit rather than concise
- Use active voice and imperative mood
- Include complete, executable examples
- Use visual markers: ✅ (allowed), ❌ (forbidden), ⚠️ (warning), 💡 (tip)
- Repeat important information when necessary

### 5.4 Security Considerations

Content MUST NOT include:
- Real API keys, tokens, or credentials (use placeholders like `YOUR_API_KEY`)
- Internal infrastructure details
- Private endpoint URLs
- Security vulnerabilities

## 6. Discoverability

### 6.1 robots.txt

Sites SHOULD reference llms.txt in `robots.txt`:

```
# AI Agent Help
User-agent: *
Allow: /.well-known/llms.txt

# AI-specific user agents
User-agent: GPTBot
User-agent: ClaudeBot
User-agent: ChatGPT-User
Allow: /
Crawl-delay: 1
```

### 6.2 HTML Meta Tag

Sites MAY include a meta tag:

```html
<meta name="llms-txt" content="/.well-known/llms.txt">
```

### 6.3 HTTP Header

Sites MAY include a custom header:

```http
X-LLMs-Txt: /.well-known/llms.txt
```

### 6.4 Link Relation

Sites MAY use a Link header or HTML `<link>` element:

```http
Link: </.well-known/llms.txt>; rel="llms-help"
```

```html
<link rel="llms-help" href="/.well-known/llms.txt">
```

## 7. Hierarchical Structure (Optional)

### 7.1 Subdirectory Files

Large sites MAY provide additional llms.txt files in subdirectories:

```
https://example.com/.well-known/llms.txt     # Root - general info
https://example.com/api/llms.txt             # API-specific
https://example.com/app/llms.txt             # Application-specific
```

### 7.2 Root File Requirements

If hierarchical files exist, the root file SHOULD:
- Include a "Site Structure" section listing subdirectory files
- Define global constraints (rate limits, authentication)

### 7.3 Inheritance

Subdirectory files:
- Inherit front matter from parent unless overridden
- May have more restrictive `acl` levels
- Should reference the root file for general context

## 8. Examples

### 8.1 Minimal Example (Pure llms.txt)

```markdown
# Acme API

> REST API for project management and team collaboration

## Documentation

- [API Reference](https://api.acme.com/docs)
- [Authentication Guide](https://api.acme.com/docs/auth)
- [Rate Limits](https://api.acme.com/docs/limits)

## Getting Started

Register at https://acme.com/signup to obtain an API key.

Use the API key in the Authorization header:

GET /api/projects HTTP/1.1
Host: api.acme.com
Authorization: Bearer YOUR_API_KEY
```

### 8.2 Extended Example (With Front Matter)

```markdown
---
acl: interact
version: 2.1.0
api_endpoint: https://api.acme.com/v2
rate_limit: 1000/hour
auth_methods: [bearer, api_key]
api_docs: https://api.acme.com/docs
cli_available: true
cli_docs: https://acme.com/cli
last_updated: 2026-01-12
---

# Acme API

> REST API for project management and team collaboration

## Authentication

Acme API uses Bearer token authentication.

**Get API Key:**
1. Register at https://acme.com/signup
2. Navigate to Settings > API Keys
3. Generate new key with desired scopes

**Use API Key:**

```http
GET /api/projects HTTP/1.1
Host: api.acme.com
Authorization: Bearer YOUR_API_KEY
Accept: application/json
```

## Rate Limits

- **Free tier**: 100 requests/hour
- **Pro tier**: 1,000 requests/hour  
- **Enterprise**: Custom limits

**Rate Limit Headers:**

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1736265600
```

## Allowed Operations

✅ AI agents MAY:
- Read public project data
- Create and update tasks
- Search and filter
- Export data (JSON)

## Forbidden Operations

❌ AI agents MUST NOT:
- Delete projects (requires human confirmation)
- Modify permissions
- Send notifications to users
- Exceed rate limits

## Error Handling

### Common Status Codes

- `200 OK` - Success
- `201 Created` - Resource created
- `400 Bad Request` - Invalid request
- `401 Unauthorized` - Missing/invalid API key
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource doesn't exist
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error (retry with backoff)

### Error Response Format

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Field 'name' is required",
    "details": {
      "field": "name",
      "constraint": "required"
    }
  }
}
```

## Quick Start

Verify access and fetch projects:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Accept: application/json" \
     https://api.acme.com/v2/projects
```

Expected response:

```json
{
  "data": [
    {
      "id": "proj_123",
      "name": "Website Redesign",
      "status": "active",
      "created_at": "2026-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "total": 42,
    "page": 1,
    "per_page": 20,
    "has_next": true
  }
}
```

## Alternative Access

**CLI**: https://acme.com/cli  
**API Docs**: https://api.acme.com/docs  
**Status**: https://status.acme.com
```

### 8.3 Read-Only Example

```markdown
---
acl: read
version: 1.0.0
---

# Company Blog

> Engineering blog and company news

## Overview

This is a public blog. AI agents may read all published content but cannot post or comment.

## Allowed Operations

✅ AI agents MAY:
- Read all published posts
- Access post metadata
- Browse archives and categories

## Forbidden Operations

❌ AI agents MUST NOT:
- Post comments
- Submit contact forms
- Attempt to access unpublished drafts

## Content Access

All posts available at:
- https://blog.example.com/posts/
- RSS feed: https://blog.example.com/feed.xml
```

## 9. Implementation Guidance

### 9.1 Static vs. Dynamic

Implementations MAY:
1. Serve static markdown files (RECOMMENDED)
2. Generate content dynamically based on user context

Static files are preferred for cacheability and transparency.

### 9.2 Platform Examples

#### Nginx

```nginx
location = /.well-known/llms.txt {
    alias /var/www/html/llms.txt;
    add_header Content-Type "text/markdown; charset=utf-8";
    add_header X-LLMs-Txt "/.well-known/llms.txt";
}

location = /llms.txt {
    return 301 /.well-known/llms.txt;
}
```

#### Express.js

```javascript
app.get('/.well-known/llms.txt', (req, res) => {
  res.type('text/markdown');
  res.header('X-LLMs-Txt', '/.well-known/llms.txt');
  res.sendFile(path.join(__dirname, 'public', 'llms.txt'));
});

app.get('/llms.txt', (req, res) => {
  res.redirect(301, '/.well-known/llms.txt');
});
```

#### Next.js

```typescript
// app/.well-known/llms.txt/route.ts
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
      'X-LLMs-Txt': '/.well-known/llms.txt',
    },
  });
}
```

### 9.3 Front Matter Parsing

Implementations that support YAML front matter should:
- Use standard YAML 1.2 parsers
- Handle missing front matter gracefully (treat as pure markdown)
- Validate field types and required fields
- Ignore unknown fields for forward compatibility

### 9.4 Testing

Implementers SHOULD:
- Test with actual AI agents (ChatGPT, Claude, etc.)
- Validate YAML parses correctly
- Verify all examples are executable
- Monitor for AI-related errors in logs
- Update content when APIs/structure changes

## 10. Security Considerations

### 10.1 Information Disclosure

Per RFC 8615 Section 4.1, applications must consider:
- Exposure of sensitive data
- Denial-of-service attack vectors
- Authentication and authorization
- Administrator awareness of `.well-known` directory contents

Sites MUST enforce technical controls (rate limiting, ACLs, authentication) rather than relying solely on documentation.

### 10.2 User Agent Identification

Sites SHOULD require AI agents to:
- Use descriptive User-Agent headers
- Identify themselves as automated agents
- Respect rate limits and Retry-After headers

Example requirement:

```markdown
## User-Agent Requirements

AI agents MUST identify themselves:

✅ GOOD:
User-Agent: MyAIBot/1.0 (+https://example.com/bot-info)

❌ BAD (pretending to be human browser):
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

### 10.3 Access Control Enforcement

The `acl` field is documentation only. Sites MUST implement technical access controls:
- Rate limiting per API key/IP
- Authentication enforcement
- Permission checking
- Audit logging

## 11. Comparison with Alternatives

### 11.1 vs. robots.txt

- **robots.txt**: Crawl permissions and delays
- **llms.txt**: How to interact (APIs, forms, schemas)
- **Relationship**: Complementary; both should be used

### 11.2 vs. sitemap.xml

- **sitemap.xml**: URL inventory for search engines
- **llms.txt**: Operational guidance for AI agents
- **Relationship**: Complementary

### 11.3 vs. OpenAPI/Swagger

- **OpenAPI**: Machine-readable API specification
- **llms.txt**: Human-readable guidance including non-API interactions (forms, navigation, policies)
- **Relationship**: llms.txt can link to OpenAPI specs

### 11.4 vs. web__ai_help Proposal

- **web__ai_help**: Comprehensive specification with detailed schemas
- **llms.txt**: Simpler, focused on single-file markdown with optional structure
- **Relationship**: This spec is a simplified subset inspired by web__ai_help

## 12. Evolution and Versioning

### 12.1 Specification Versioning

This specification uses semantic versioning:
- **MAJOR**: Incompatible changes to required fields or file location
- **MINOR**: New optional fields or recommended sections
- **PATCH**: Clarifications and examples

### 12.2 Content Versioning

Individual sites SHOULD version their content using the `version` front matter field.

### 12.3 Forward Compatibility

Parsers MUST:
- Ignore unknown front matter fields
- Treat missing front matter as valid (pure markdown)
- Degrade gracefully for unsupported features

## 13. References

- [llms.txt](https://llmstxt.org/) - Original llms.txt specification
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615.html) - Well-Known URIs
- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for RFCs
- [CommonMark](https://commonmark.org/) - Markdown specification
- [YAML 1.2](https://yaml.org/spec/1.2.2/) - YAML specification
- [dashdash web__ai_help](https://github.com/visionik/dashdash) - Inspiration for extended capabilities

## 14. Acknowledgments

We are deeply grateful to the team behind [llmstxt.org](https://llmstxt.org/) for their pioneering work in creating a simple, elegant standard for LLM-friendly documentation. Their insight that "markdown is human and LLM readable, but is also in a precise format allowing fixed processing methods" has provided a crucial foundation for making web content more accessible to AI agents.

The original llms.txt specification's focus on simplicity and human readability is exemplary, and this proposal attempts to honor that spirit while exploring additional capabilities for complex interactive applications.

This document also builds upon:
- [llmstxt.org](https://llmstxt.org/) - The foundational llms.txt convention that inspired this work
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615.html) - IETF Well-Known URIs standard for site-wide metadata
- [dashdash project](https://github.com/visionik/dashdash) - web__ai_help proposal for interactive web applications
- Real-world AI agent interaction patterns observed in production systems

**We invite the llmstxt.org team and the broader community to provide feedback, critique, and alternative approaches.** This proposal is meant to start a conversation, not to dictate a solution.
