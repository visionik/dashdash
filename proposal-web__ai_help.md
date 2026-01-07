# Web AI Help Specification

**Version:** 1.0.0  
**Status:** Draft  
**Last Updated:** 2026-01-07

## 1. Introduction

This document specifies a standard approach for websites and web applications to provide AI-optimized usage guidance through a dedicated `.well-known/__ai_help` file (with optional subdirectory `__ai_help.md` files). As Large Language Models (LLMs) and AI agents increasingly interact with web services, specialized documentation format requirements emerge that differ from traditional human-oriented documentation.

### 1.1 Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## 2. Rationale

### 2.1 Problem Statement

Traditional web documentation (FAQs, user guides, API docs) is optimized for human navigation with visual hierarchy, tutorials, and conversational tone. AI agents have different needs:

- **Structured endpoint discovery** - Clear mapping of URLs to capabilities
- **Rate limiting and quotas** - Explicit constraints on AI usage
- **Authentication flows** - Step-by-step credential management
- **Data format specifications** - Comprehensive input/output schemas
- **Error code mapping** - Common failure modes and solutions
- **Permission boundaries** - What AIs can and cannot do
- **Alternative access methods** - CLI, API, or MCP alternatives
- **Form field specifications** - Exact expected formats and validation rules

### 2.2 Benefits

- **Improved AI accuracy** - Reduces hallucination and incorrect web interactions
- **Reduced server load** - AIs can understand constraints before making requests
- **Standardization** - Common filename creates predictable discovery pattern
- **Separation of concerns** - Keeps human docs clean while providing AI detail
- **Progressive disclosure** - Hierarchical files for complex site structures
- **Access control clarity** - Explicit AI permissions and boundaries

## 3. Specification

### 3.1 File Naming and Location

#### 3.1.1 File Naming Convention

**Root File:**
Websites MUST use `.well-known/__ai_help` for the root file.

**Rationale:**
- Follows RFC 8615 standard for `.well-known` URIs
- Retains `__ai_help` naming pattern for consistency
- No file extension (following `.well-known` convention, though content is markdown)
- Standard, predictable location for AI agents

**Subdirectory Files:**
Websites MUST use `__ai_help.md` (double underscore prefix, `.md` extension) for subdirectory files.

**Rationale:**
- Double underscore prefix indicates meta/system file (common in many frameworks)
- `.md` extension ensures readable plain text when viewed directly
- `.well-known` is typically root-only, so subdirectories use the full filename pattern

#### 3.1.2 Root Location

Sites MUST implement the root file at `.well-known/__ai_help` (following RFC 8615):
```
https://example.com/.well-known/__ai_help
```

**Note:** No file extension for the root file to follow `.well-known` URI conventions, though content is markdown.

#### 3.1.3 Hierarchical Structure

Sites MAY implement additional `__ai_help.md` files in subdirectories for complex applications:
```
https://example.com/.well-known/__ai_help      # Root - general site info (RFC 8615)
https://example.com/app/__ai_help.md           # Application-specific
https://example.com/app/dashboard/__ai_help.md # Dashboard-specific
https://example.com/app/inventory/__ai_help.md # Inventory-specific
https://example.com/api/__ai_help.md           # API-specific guidance
```

**Rationale for hybrid approach:**
- Root file uses `.well-known/__ai_help` for RFC 8615 compliance and standardization
- Subdirectory files use `__ai_help.md` pattern since `.well-known` is typically root-only
- The `__` prefix is consistent across all files for recognizability

When hierarchical files exist, the root file MUST indicate this with `sub: true` in front matter (see §3.2).

### 3.2 File Format

#### 3.2.1 Structure

Files MUST be markdown with YAML front matter (following the dashdash web format specification).

#### 3.2.2 Required Front Matter

The front matter MUST include:

```yaml
---
abt: https://github.com/visionik/dashdash
sub: true
ver: https://github.com/visionik/dashdash/blob/main/WEB-AI-HELP-SPEC.md#v1.0.0
acl: read
cli: https://example.com/cli
mcp: none
api: https://example.com/api/docs
---
```

**Field Definitions:**

- **abt** (REQUIRED): Link to the dashdash GitHub repository
- **sub** (REQUIRED): Boolean indicating whether subdirectory `__ai_help.md` files exist
- **ver** (REQUIRED): Link to the version of this specification the file implements
- **acl** (REQUIRED): AI access control level (see §3.3)
- **cli** (REQUIRED): URL to CLI documentation, or `none` if no CLI exists
- **mcp** (REQUIRED): URL to MCP server documentation, or `none` if no MCP exists
- **api** (REQUIRED): URL to API documentation, or `none` if no API exists

#### 3.2.3 Content Format

Following the front matter, content MUST be markdown with:
- Clear section headers (using `##` and `###`)
- Code blocks with appropriate language tags
- Lists for enumeration
- Emphasis for warnings (e.g., ⚠️, ❌, ✅ emoji)

### 3.3 Access Control Levels

The `acl` field MUST be one of:

- **none** - AI agents MUST NOT interact with this site
- **read** - AI agents MAY read public information only
- **browse** - AI agents MAY navigate and search but not submit forms
- **interact** - AI agents MAY submit forms and perform actions (with constraints in content)
- **full** - AI agents have same access as authenticated users (subject to rate limits)

Sites SHOULD default to `read` unless explicitly designed for AI interaction.

### 3.4 Discoverability

#### 3.4.1 robots.txt Integration

Sites SHOULD reference the AI help file in their `robots.txt`:

```
# AI Agent Help
User-agent: *
Allow: /.well-known/__ai_help
Allow: /__ai_help.md

# AI-specific user agents
User-agent: GPTBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
Allow: /
Crawl-delay: 1
```

#### 3.4.2 HTML Meta Tags

Sites MAY include a meta tag in their HTML `<head>`:

```html
<meta name="ai-help" content="/.well-known/__ai_help">
```

#### 3.4.3 HTTP Headers

Sites MAY include a custom HTTP header on responses:

```
X-AI-Help: /.well-known/__ai_help
```

#### 3.4.4 Legacy Redirect

For backward compatibility, sites MAY provide a redirect from `/__ai_help.md` to `/.well-known/__ai_help`.

### 3.5 Content Requirements

#### 3.5.1 Required Sections

The help content MUST include:

1. **Overview** - Site/app purpose and primary capabilities
2. **Access Requirements** - Authentication, account creation, prerequisites
3. **Site Structure** - Main sections and their purposes
4. **URL Patterns** - Key endpoint patterns and routing
5. **Rate Limits and Performance** (§3.5.10) - Request quotas, throttling, timeouts
6. **Allowed Operations** - What AIs are permitted to do
7. **Forbidden Operations** - What AIs MUST NOT do
8. **Error Handling** - Common HTTP errors and their meanings

#### 3.5.2 Recommended Sections

The help content SHOULD include:

1. **Quick Start** - Minimal viable interaction to verify access
2. **Authentication Guide** (§3.5.5) - Step-by-step auth flows
3. **Form Specifications** (§3.5.4) - Field names, formats, validation rules
4. **Data Formats** - Input/output schemas (JSON, form-encoded, etc.)
5. **Search Patterns** - How to query/filter data
6. **Navigation Guide** - How to traverse the site programmatically
7. **Troubleshooting** - Common errors with solutions
8. **Best Practices** - AI-specific recommendations
9. **Alternative Access** - Links to CLI, API, or MCP alternatives
10. **Examples** - Common use cases with actual requests
11. **Webhooks/Callbacks** - If applicable, how to receive async notifications
12. **CORS/Security** - Cross-origin policies relevant to AI access
13. **Input Schema** (§3.5.6) - Request body and parameter schemas
14. **Output Schema** (§3.5.7) - Response structures and guarantees
15. **Operational Semantics** (§3.5.8) - Idempotency, side effects, pagination
16. **Capabilities and Categories** (§3.5.9) - Endpoint categorization
17. **Lifecycle Metadata** (§3.5.11) - Versioning, deprecations, sunset schedules

#### 3.5.3 Date/Time Handling

If the site accepts date or time parameters (in forms, queries, etc.), the documentation MUST:
- List ALL supported formats with examples
- List common UNSUPPORTED formats with explanations
- Include timezone handling guidance
- Show how dates appear in URLs vs. form data

Example:
```markdown
### ✅ SUPPORTED DATE FORMATS

**In URL query parameters:**
- `?date=2026-01-07` - ISO 8601 date
- `?date=2026-01-07T10:30:00Z` - ISO 8601 datetime with UTC
- `?from=2026-01-01&to=2026-01-31` - Date ranges

**In form fields:**
- `<input name="start_date" value="2026-01-07">` - ISO 8601

### ❌ UNSUPPORTED FORMATS

- `?date=01/07/2026` - Ambiguous MM/DD/YYYY or DD/MM/YYYY
- `?date="7 days ago"` - Relative dates not supported
```

#### 3.5.4 Form Field Specifications

For sites with forms, documentation MUST include:
- Field names (exact HTML `name` attributes)
- Expected data types and formats
- Validation rules (min/max length, patterns, etc.)
- Required vs. optional fields
- Default values if any
- Submit URLs and methods (POST/PUT/etc.)

Example:
```markdown
### User Registration Form

**URL:** `POST https://example.com/api/register`

**Content-Type:** `application/json`

**Fields:**
- `email` (REQUIRED, string): Valid email address, max 255 chars
- `password` (REQUIRED, string): Min 8 chars, must contain uppercase, lowercase, digit
- `name` (OPTIONAL, string): Display name, max 100 chars
- `agree_terms` (REQUIRED, boolean): Must be `true`

**Example Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123",
  "name": "John Doe",
  "agree_terms": true
}
```

**Success Response:** `201 Created`
**Error Responses:** `400 Bad Request`, `409 Conflict` (email exists)
```

#### 3.5.5 Authentication Flows

Documentation MUST detail authentication mechanisms:

```markdown
### Authentication Methods

#### 1. API Key (RECOMMENDED for AI agents)

**Obtain key:**
1. Register at https://example.com/register
2. Navigate to https://example.com/settings/api
3. Click "Generate API Key"
4. Store key securely

**Usage:**
```http
GET /api/data HTTP/1.1
Host: example.com
Authorization: Bearer YOUR_API_KEY_HERE
```

#### 2. OAuth 2.0 (for delegated access)

**Not recommended for AI agents** - requires interactive browser flow

#### 3. Session Cookies

**For web scraping scenarios only:**
1. POST credentials to `/login`
2. Store `Set-Cookie` header
3. Include cookie in subsequent requests
```

#### 3.5.6 Input Schema

For sites with structured API endpoints or forms, documentation SHOULD include schema specifications:

**Requirements:**
- Request body schemas (JSON, form-data, multipart)
- Query parameter specifications
- Path parameter formats
- Header requirements
- Field types and validation rules
- Required vs. optional fields
- Mutually exclusive parameters

Example:
```markdown
## API Input Schema

### Endpoint: `POST /api/projects`

**Request Schema (JSON):**
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
      "pattern": "^[a-zA-Z0-9-_\\s]+$",
      "description": "Project name"
    },
    "type": {
      "type": "string",
      "enum": ["public", "private", "internal"],
      "description": "Visibility level"
    },
    "tags": {
      "type": "array",
      "items": {"type": "string"},
      "maxItems": 10,
      "description": "Metadata tags"
    },
    "metadata": {
      "type": "object",
      "additionalProperties": {"type": "string"},
      "description": "Custom key-value pairs"
    }
  }
}
```

**Required Headers:**
```http
Content-Type: application/json
Authorization: Bearer YOUR_API_KEY
```

**Query Parameters:**
- `idempotency_key` (optional, string): Unique key for idempotent creation

**Constraints:**
- `type: private` requires authenticated user
- `tags` array items must be unique
- `metadata` keys cannot start with underscore
```

#### 3.5.7 Output Schema

For API endpoints, documentation SHOULD specify response structures:

**Requirements:**
- Success response schemas by status code
- Error response format
- Optional vs. guaranteed fields
- Pagination structure
- Header information

Example:
```markdown
## API Output Schema

### Endpoint: `GET /api/projects`

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": "string (UUID v4)",
      "name": "string",
      "type": "string (enum: public|private|internal)",
      "status": "string (enum: active|archived|deleted)",
      "created_at": "string (ISO 8601)",
      "updated_at": "string (ISO 8601, nullable)",
      "owner": {
        "id": "string (UUID)",
        "username": "string",
        "avatar_url": "string (URL, nullable)"
      },
      "tags": ["string"],
      "metadata": "object (optional)"
    }
  ],
  "pagination": {
    "total": "integer",
    "page": "integer",
    "per_page": "integer",
    "total_pages": "integer",
    "has_next": "boolean",
    "has_prev": "boolean",
    "next_url": "string (URL, nullable)",
    "prev_url": "string (URL, nullable)"
  },
  "meta": {
    "timestamp": "string (ISO 8601)",
    "request_id": "string"
  }
}
```

**Response Headers:**
```http
Content-Type: application/json; charset=utf-8
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1736265600
X-Request-ID: abc123def456
```

**Error Response (4xx/5xx):**
```json
{
  "error": {
    "code": "string (e.g., INVALID_REQUEST, UNAUTHORIZED)",
    "message": "string (human-readable)",
    "details": {
      "field": "string (optional)",
      "constraint": "string (optional)",
      "provided": "any (optional)"
    },
    "request_id": "string",
    "timestamp": "string (ISO 8601)"
  }
}
```

**Field Guarantees:**
- `id`, `name`, `type`, `status`, `created_at` always present
- `updated_at` null if never modified
- `owner.avatar_url` null if not set
- `metadata` only present if custom fields exist
- `pagination.next_url`/`prev_url` null at boundaries
```

#### 3.5.8 Operational Semantics

Documentation SHOULD specify HTTP operational behavior:

**Requirements:**
- Idempotency guarantees by HTTP method
- Side effects of operations
- Pagination mechanisms
- Partial success handling
- Retry safety
- CORS behavior

Example:
```markdown
## Operational Semantics

### HTTP Method Semantics

**Idempotent Methods (Safe to Retry):**
- `GET`, `HEAD`, `OPTIONS` - Read-only, no side effects
- `PUT` - Replaces entire resource, repeatable
- `DELETE` - Idempotent (returns success even if already deleted)

**Idempotent with Key:**
- `POST` with `Idempotency-Key` header - Safe to retry within 24 hours

**Non-Idempotent:**
- `POST` without idempotency key - May create duplicates
- `PATCH` on increment/decrement operations

### Idempotency Keys

**Usage:**
```http
POST /api/projects HTTP/1.1
Idempotency-Key: unique-key-123
Content-Type: application/json

{"name": "My Project"}
```

**Behavior:**
- First request: Creates resource, returns `201 Created`
- Retry within 24h: Returns original `201` response from cache
- After 24h: Key expires, new resource may be created

### Side Effects by Endpoint

| Endpoint | Method | Side Effects | Reversible? |
|----------|--------|-------------|-------------|
| `/api/projects` | GET | None | N/A |
| `/api/projects` | POST | Creates resource, sends webhook, logs audit | Manual DELETE |
| `/api/projects/{id}` | PUT | Updates resource, triggers reindex | Previous version in history |
| `/api/projects/{id}` | DELETE | Soft-delete (30-day retention) | `POST /api/projects/{id}/restore` |
| `/api/export` | POST | Async job creation, email notification | Cannot cancel |

### Pagination

**Cursor-Based (Recommended):**
```http
GET /api/projects?limit=50 HTTP/1.1

Response:
{
  "data": [...],
  "pagination": {
    "next_url": "/api/projects?limit=50&cursor=abc123"
  }
}
```

**Benefits:**
- Stable across concurrent modifications
- Efficient for large datasets
- No duplicate/missing items

**Offset-Based (Deprecated):**
```http
GET /api/projects?page=2&per_page=50 HTTP/1.1
```

⚠️ May return duplicates or skip items if data changes between requests

### Partial Success

**Batch Endpoints:**
```http
POST /api/projects/bulk-delete HTTP/1.1
Content-Type: application/json

{"ids": ["id1", "id2", "id3"]}
```

**Response (207 Multi-Status):**
```json
{
  "results": [
    {"id": "id1", "status": 200, "message": "Deleted"},
    {"id": "id2", "status": 404, "message": "Not found"},
    {"id": "id3", "status": 200, "message": "Deleted"}
  ],
  "summary": {
    "total": 3,
    "succeeded": 2,
    "failed": 1
  }
}
```

### CORS Behavior

**Allowed Origins:**
- `https://*.example.com` - All subdomains
- `http://localhost:*` - Local development

**Preflight Response:**
```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, Idempotency-Key
Access-Control-Max-Age: 86400
```
```

#### 3.5.9 Capabilities and Categories

Documentation SHOULD categorize endpoints for improved discovery:

**Requirements:**
- Functional categories/domains
- Operation types (read/write/admin)
- Resource types
- Common use cases

Example:
```markdown
## API Capabilities

### By Category

**Resource Management (CRUD):**
- `GET /api/projects` - List projects
- `POST /api/projects` - Create project  
- `GET /api/projects/{id}` - Get project
- `PUT /api/projects/{id}` - Update project
- `DELETE /api/projects/{id}` - Delete project
- Tags: `crud`, `projects`, `core`

**Search and Discovery:**
- `GET /api/search` - Full-text search
- `GET /api/projects?filter[status]=active` - Filtered lists
- Tags: `search`, `query`, `discovery`

**Data Export:**
- `POST /api/export` - Create export job
- `GET /api/export/{id}` - Check export status
- `GET /api/export/{id}/download` - Download export
- Tags: `export`, `data`, `compliance`

**Administration:**
- `GET /api/admin/users` - List users
- `POST /api/admin/audit` - Query audit log
- Tags: `admin`, `security`, `management`

**Webhooks:**
- `GET /api/webhooks` - List webhooks
- `POST /api/webhooks` - Create webhook
- `POST /api/webhooks/{id}/test` - Test webhook
- Tags: `webhooks`, `integration`, `events`

### By Operation Type

**Read Operations (Safe, Cacheable):**
- `GET /api/projects`
- `GET /api/projects/{id}`
- `GET /api/search`

**Write Operations (Idempotent with key):**
- `POST /api/projects` (with `Idempotency-Key`)
- `PUT /api/projects/{id}`

**Destructive Operations (Time-limited reversibility):**
- `DELETE /api/projects/{id}` (30-day soft delete)
- `POST /api/projects/{id}/purge` (immediate, irreversible)

**Admin Operations (Elevated permissions):**
- `POST /api/admin/*`
- `DELETE /api/admin/users/{id}`

### By Resource Type

- **Projects**: `/api/projects/*`
- **Users**: `/api/users/*`, `/api/admin/users/*`
- **Teams**: `/api/teams/*`
- **Webhooks**: `/api/webhooks/*`
- **Exports**: `/api/export/*`
```

#### 3.5.10 Rate Limits and Performance

Documentation MUST specify rate limits and performance characteristics:

**Requirements:**
- Rate limit tiers by plan
- HTTP headers for rate limiting
- Timeout values
- Payload size limits
- Concurrent request limits
- Caching directives

Example:
```markdown
## Rate Limits and Performance

### Rate Limits

**By Plan Tier:**
| Tier | Requests/Hour | Burst (per minute) | Concurrent |
|------|---------------|-------------------|------------|
| Free | 100 | 10 | 2 |
| Pro | 1,000 | 100 | 10 |
| Enterprise | 10,000 | 1,000 | 50 |

**Rate Limit Headers:**
All API responses include:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950  
X-RateLimit-Reset: 1736265600
Retry-After: 60
```

**When Rate Limited (429 Too Many Requests):**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Retry after 60 seconds.",
    "details": {
      "limit": 1000,
      "window": "1 hour",
      "retry_after": 60
    }
  }
}
```

**Best Practices:**

💡 **Check rate limit headers proactively:**
```python
if int(response.headers['X-RateLimit-Remaining']) < 10:
    time.sleep(60)  # Back off before limit
```

💡 **Respect Retry-After header:**
```python
if response.status_code == 429:
    retry_after = int(response.headers.get('Retry-After', 60))
    time.sleep(retry_after)
```

💡 **Use batch endpoints when available:**
```http
# Good: Single request
POST /api/projects/bulk-create
{"projects": [{...}, {...}, {...}]}

# Avoid: Multiple requests
POST /api/projects {...}
POST /api/projects {...}
POST /api/projects {...}
```

### Timeouts

**Server-Side Timeouts:**
- Connection: 10 seconds
- Request processing: 30 seconds
- Long-running operations: Use async pattern

**Long-Running Operations:**
```http
POST /api/export HTTP/1.1

Response (202 Accepted):
{
  "job_id": "job_abc123",
  "status": "pending",
  "status_url": "/api/export/job_abc123"
}

GET /api/export/job_abc123 HTTP/1.1

Response (200 OK):
{
  "job_id": "job_abc123",
  "status": "completed",
  "download_url": "/api/export/job_abc123/download",
  "expires_at": "2026-01-08T00:00:00Z"
}
```

### Payload Limits

**Request Limits:**
- JSON body: 10 MB
- File upload: 100 MB per file
- Batch operations: 1,000 items per request
- URL length: 8,192 characters

**Response Limits:**
- JSON response: 50 MB (use pagination for large datasets)
- File download: No limit (streamed)

**Recommended Page Sizes:**
- Default: 50 items
- Maximum: 1,000 items
- Suggested: 100-200 items for optimal performance

### Caching

**Cache Headers:**
```http
Cache-Control: public, max-age=300
ETag: "abc123def456"
Last-Modified: Mon, 06 Jan 2026 12:00:00 GMT
```

**Conditional Requests:**
```http
GET /api/projects/proj_123 HTTP/1.1
If-None-Match: "abc123def456"

Response: 304 Not Modified (if unchanged)
```

**Caching Strategy:**
- `GET /api/projects`: Cache 60 seconds
- `GET /api/projects/{id}`: Cache 5 minutes, support ETag
- `GET /api/admin/*`: No caching (`Cache-Control: no-store`)
- `POST/PUT/DELETE`: Invalidate related caches
```

#### 3.5.11 Lifecycle Metadata

Documentation SHOULD include versioning and deprecation information:

**Requirements:**
- API version information
- Deprecated endpoints
- Breaking change notices
- Migration guides
- Sunset schedules

Example:
```markdown
## API Lifecycle and Versioning

### Current Version

**API Version:** v2.4  
**Base URL:** `https://api.example.com/v2`
**Status:** ✅ Stable

**Version Headers:**
```http
GET /api/projects HTTP/1.1
Accept: application/vnd.example.v2+json

Response:
API-Version: 2.4
Deprecation: false
```

### Deprecated Endpoints

⚠️ **Deprecated in v2.3 (Sunset: 2026-06-01):**

| Deprecated Endpoint | Replacement | Reason |
|--------------------|-------------|--------|
| `GET /api/v2/users?visibility=V` | `GET /api/v2/users?type=V` | Parameter naming consistency |
| `GET /api/v2/list?offset=N` | `GET /api/v2/projects?cursor=TOKEN` | Unstable pagination |
| `POST /api/v2/batch` | `POST /api/v2/projects/bulk-create` | Resource-specific batching |

**Deprecation Headers:**
```http
Deprecation: true
Sunset: Sat, 01 Jun 2026 00:00:00 GMT
Link: </api/v2/projects>; rel="successor-version"
```

**Migration Example:**

❌ **Old (deprecated):**
```http
GET /api/v2/list?offset=20 HTTP/1.1
```

✅ **New:**
```http
GET /api/v2/projects?limit=20&cursor=abc123 HTTP/1.1
```

### Breaking Changes by Version

**v2.0.0 (Released: 2025-01-01):**
- Changed default response format to JSON (was XML)
- Removed legacy `/api/v1` endpoints
- Required `Authorization` header for all endpoints
- Changed date format to ISO 8601 (was Unix timestamp)

**v1.x → v2.0 Migration:**
```bash
# Update base URL
- https://api.example.com/v1/...
+ https://api.example.com/v2/...

# Update auth header
- X-API-Key: YOUR_KEY
+ Authorization: Bearer YOUR_KEY

# Update date handling
- "created": 1609459200
+ "created_at": "2021-01-01T00:00:00Z"
```

### Version Support Schedule

| Version | Release Date | Sunset Date | Status | Action Required |
|---------|-------------|-------------|--------|----------------|
| v1.x | 2023-01-01 | 2025-06-01 | ❌ EOL | Upgrade to v2.x |
| v2.x | 2025-01-01 | 2027-01-01 | ✅ Stable | Current recommended |
| v3.x | TBD | TBD | 🚧 Beta | Preview at api-beta.example.com |

**Support Policy:**
- **Stable versions**: 24 months from release
- **Security fixes**: Current version only
- **Bug fixes**: Current + previous version
- **New features**: Current version only
- **Deprecation notice**: Minimum 6 months before sunset

### Sunset Process

**6 months before sunset:**
- Deprecation headers added to affected endpoints
- Email notification to API key owners
- Banner in developer portal

**3 months before:**
- Warning logs in response
- Second email notification

**1 month before:**
- Rate limit reduced to 50% for deprecated endpoints
- Final email notification

**Sunset date:**
- Endpoints return `410 Gone`
- Response includes migration guide URL
```

#### 2. OAuth 2.0 (for delegated access)

**Not recommended for AI agents** - requires interactive browser flow

#### 3. Session Cookies

**For web scraping scenarios only:**
1. POST credentials to `/login`
2. Store `Set-Cookie` header
3. Include cookie in subsequent requests
```

### 3.6 Content Style

#### 3.6.1 Verbosity

AI help content SHOULD be more verbose than human documentation. Explicit is better than concise.

#### 3.6.2 Tone

Content SHOULD:
- Be direct and imperative
- Use active voice
- Avoid ambiguity
- Repeat important information if needed
- Clearly distinguish between capabilities and constraints

#### 3.6.3 Visual Markers

Content SHOULD use emoji or symbols to highlight:
- ✅ Allowed actions
- ❌ Forbidden actions
- ⚠️ Important warnings
- 💡 Tips and best practices
- 🔧 Troubleshooting steps
- 🔒 Security considerations

## 4. Hierarchical Behavior

### 4.1 Inheritance

When subdirectory `__ai_help.md` files exist:
- Front matter fields from parent directories are inherited
- Content sections can be overridden or extended
- More specific rules take precedence

### 4.2 Root File Requirements

The root `__ai_help.md` MUST:
- Set `sub: true` if subdirectory files exist
- Provide a site map or directory structure showing where sub-files are located
- Define global constraints (rate limits, authentication)

Example:
```markdown
## Site Structure

This site has AI help documentation at multiple levels:

- `/.well-known/__ai_help` - General site information (this file)
- `/app/__ai_help.md` - Main application features
- `/app/dashboard/__ai_help.md` - Dashboard-specific operations
- `/app/inventory/__ai_help.md` - Inventory management features
- `/api/__ai_help.md` - API-specific guidance

Each subdirectory file inherits and extends the rules from parent directories.
```

### 4.3 Subdirectory Files

Subdirectory `__ai_help.md` files SHOULD:
- Reference the root file for general context
- Focus on section-specific operations
- Override only the front matter fields that change (e.g., more restrictive `acl`)

## 5. Implementation Guidance

### 5.1 Static vs. Dynamic

Implementations MAY either:
1. Serve static markdown files (RECOMMENDED for simplicity)
2. Generate content dynamically based on user authentication/context

Static files are preferred for cacheability and transparency.

### 5.2 Maintenance

The AI help content SHOULD:
- Be updated whenever site structure or API changes
- Include version information in front matter
- Be reviewed during UI/UX changes
- Track common AI mistakes and address them in documentation

### 5.3 Testing

Implementers SHOULD:
- Validate YAML front matter parses correctly
- Verify markdown renders properly
- Test that examples are accurate and executable
- Use actual AI agents to validate guidance
- Monitor for AI-related errors in logs

### 5.4 Examples by Platform

#### 5.4.1 Static Site (Nginx)

```nginx
location = /__ai_help.md {
    root /var/www/html;
    add_header Content-Type text/markdown;
    add_header X-AI-Help /__ai_help.md;
}
```

#### 5.4.2 Express.js

```javascript
app.get('/__ai_help.md', (req, res) => {
    res.type('text/markdown');
    res.header('X-AI-Help', '/__ai_help.md');
    res.sendFile(path.join(__dirname, 'public', '__ai_help.md'));
});
```

#### 5.4.3 Django

```python
from django.http import HttpResponse
from django.views import View
import os

class AIHelpView(View):
    def get(self, request):
        file_path = os.path.join(settings.BASE_DIR, '__ai_help.md')
        with open(file_path, 'r') as f:
            content = f.read()
        response = HttpResponse(content, content_type='text/markdown')
        response['X-AI-Help'] = '/__ai_help.md'
        return response
```

#### 5.4.4 Next.js

```typescript
// pages/__ai_help.md.ts
import fs from 'fs';
import path from 'path';

export default function AIHelp({ content }: { content: string }) {
    return <pre>{content}</pre>;
}

export async function getStaticProps() {
    const filePath = path.join(process.cwd(), '__ai_help.md');
    const content = fs.readFileSync(filePath, 'utf8');
    return { props: { content } };
}
```

## 6. Security Considerations

### 6.1 Information Disclosure

AI help content SHOULD NOT include:
- API keys, tokens, or credentials (even examples - use placeholders like `YOUR_API_KEY_HERE`)
- Internal system architecture or infrastructure details
- Sensitive business logic or algorithms
- Private endpoint URLs not intended for public access
- Rate limits so precise they enable abuse
- Security vulnerabilities or known exploits

### 6.2 Access Control

Sites MUST:
- Enforce `acl` restrictions through technical controls, not just documentation
- Implement rate limiting for AI user agents
- Monitor for abuse patterns
- Log AI agent interactions for audit

### 6.3 User Agent Identification

Sites SHOULD require AI agents to:
- Use descriptive User-Agent headers
- Identify themselves as automated agents
- Respect rate limits and `Retry-After` headers

Example policy:
```markdown
## User-Agent Requirements

AI agents MUST identify themselves with a User-Agent header:

✅ GOOD:
```http
User-Agent: MyAIAgent/1.0 (contact@example.com)
```

❌ BAD (pretending to be a browser):
```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```
```

## 7. Evolution and Versioning

### 7.1 Specification Versioning

This specification uses semantic versioning (MAJOR.MINOR.PATCH):
- MAJOR: Incompatible changes to front matter or file structure
- MINOR: New recommended sections or practices
- PATCH: Clarifications and examples

### 7.2 Content Versioning

Individual sites MAY version their AI help content. RECOMMENDED format in front matter:

```yaml
---
abt: https://github.com/visionik/dashdash
sub: false
ver: https://github.com/visionik/dashdash/blob/main/WEB-AI-HELP-SPEC.md#v1.0.0
acl: read
cli: none
mcp: none
api: https://api.example.com/docs
content_version: 2.1.0
last_updated: 2026-01-07
---
```

## 8. Future Considerations

### 8.1 Machine-Readable Format

Future versions MAY specify:
- JSON-LD schema for structured site metadata
- OpenAPI-like specifications for web interactions
- Formal schema for form validation rules

### 8.2 Interactive Discovery

Future versions MAY specify:
- Programmatic querying of capabilities
- Dynamic generation based on user permissions
- Real-time validation endpoints

### 8.3 Multi-language Support

Future versions MAY specify:
- Language negotiation via HTTP headers
- Multiple `__ai_help.{lang}.md` files
- Content localization guidelines

## 9. References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for use in RFCs
- [CommonMark](https://commonmark.org/) - Markdown specification
- [YAML](https://yaml.org/) - YAML specification
- [robots.txt](https://www.robotstxt.org/) - Robots exclusion standard
- [CLI AI Help Specification](https://github.com/visionik/dashdash/blob/main/README.md) - Parallel CLI standard

## 10. Acknowledgments

This specification is informed by:
- Real-world LLM interactions with web services
- Common patterns in AI agent error logs
- The CLI `--ai-help` specification
- Web scraping best practices
- API documentation standards (OpenAPI, API Blueprint)

## Appendix A: Complete Example

### Root File: `https://example.com/.well-known/__ai_help`

```markdown
---
abt: https://github.com/visionik/dashdash
sub: true
ver: https://github.com/visionik/dashdash/blob/main/WEB-AI-HELP-SPEC.md#v1.0.0
acl: interact
cli: https://example.com/cli
mcp: none
api: https://api.example.com/docs
content_version: 1.0.0
last_updated: 2026-01-07
---

# Example.com AI Help Guide

## Overview

Example.com is a project management platform that allows teams to track tasks, collaborate, and report progress. AI agents can interact with the platform to read data, create tasks, and update statuses.

## Access Requirements

### Authentication

AI agents MUST authenticate using API keys:

1. **Obtain API Key:**
   - Register at https://example.com/register
   - Navigate to Settings > API Keys
   - Generate a new key with appropriate scopes

2. **Use API Key:**
   ```http
   GET /api/projects HTTP/1.1
   Host: example.com
   Authorization: Bearer YOUR_API_KEY_HERE
   ```

### Rate Limits

- **Free tier:** 100 requests/hour
- **Pro tier:** 1000 requests/hour
- **Enterprise:** Custom limits

Rate limit headers included in responses:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1736265600
```

## Site Structure

This site has AI help documentation at multiple levels:

- `/.well-known/__ai_help` - This file (general site information)
- `/app/__ai_help.md` - Main application features
- `/api/__ai_help.md` - API-specific details

## URL Patterns

### Main Sections

- `https://example.com/` - Public homepage
- `https://example.com/app/` - Main application (requires auth)
- `https://example.com/app/projects/` - Project list
- `https://example.com/app/projects/{id}/` - Individual project
- `https://example.com/app/tasks/` - Task management
- `https://example.com/api/` - REST API endpoints

### API Endpoints

- `GET /api/projects` - List all accessible projects
- `POST /api/projects` - Create new project
- `GET /api/projects/{id}` - Get project details
- `PATCH /api/projects/{id}` - Update project
- `GET /api/projects/{id}/tasks` - List project tasks
- `POST /api/projects/{id}/tasks` - Create task in project

## Allowed Operations

✅ AI agents MAY:
- Read public project information
- Create tasks in projects they have access to
- Update task statuses
- Search and filter tasks
- Export data in JSON format
- Subscribe to webhook notifications

## Forbidden Operations

❌ AI agents MUST NOT:
- Delete projects or tasks (requires human confirmation)
- Modify user permissions or roles
- Access other users' private data
- Send notifications or emails to users
- Make purchases or modify billing
- Exceed rate limits (will result in 429 responses)

## Error Handling

### Common HTTP Status Codes

- `200 OK` - Successful request
- `201 Created` - Resource successfully created
- `400 Bad Request` - Invalid request format or parameters
- `401 Unauthorized` - Missing or invalid API key
- `403 Forbidden` - Valid auth but insufficient permissions
- `404 Not Found` - Resource doesn't exist
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error (retry with exponential backoff)

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

## Date/Time Formats

### ✅ SUPPORTED FORMATS

All dates/times must be ISO 8601:

- `2026-01-07` - Date only
- `2026-01-07T15:30:00Z` - UTC datetime
- `2026-01-07T10:30:00-05:00` - Datetime with timezone offset

### ❌ UNSUPPORTED FORMATS

- `01/07/2026` - Ambiguous locale-dependent format
- `7 Jan 2026` - Natural language dates
- `1736265600` - Unix timestamps (must convert to ISO 8601)

## Best Practices for AI Agents

💡 **Always use JSON format:**
```http
Accept: application/json
Content-Type: application/json
```

💡 **Include descriptive User-Agent:**
```http
User-Agent: MyAIAgent/1.0 (+https://mysite.com/bot-info)
```

💡 **Handle rate limits gracefully:**
- Check `X-RateLimit-Remaining` header
- Respect `Retry-After` when receiving 429
- Implement exponential backoff

💡 **Prefer API over web scraping:**
- API is faster, more reliable, and explicitly supported
- See https://api.example.com/docs for full API reference

## Alternative Access Methods

### CLI

A command-line interface is available for complex workflows:
- Installation: https://example.com/cli
- AI help: `example-cli --ai-help`

### MCP

Not currently available. Check back for future updates.

## Quick Start Example

Verify access and fetch your projects:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Accept: application/json" \
     https://example.com/api/projects
```

Expected response:
```json
{
  "projects": [
    {
      "id": "proj_123",
      "name": "Website Redesign",
      "status": "active",
      "created_at": "2026-01-01T00:00:00Z"
    }
  ]
}
```

## Troubleshooting

### Error: "Invalid API Key"

**Cause:** API key is malformed or expired

**Solution:**
1. Verify key is copied correctly (no extra spaces)
2. Check key hasn't been revoked in Settings
3. Generate new key if needed

### Error: "Rate Limit Exceeded"

**Cause:** Too many requests in time window

**Solution:**
```python
import time

if response.status_code == 429:
    retry_after = int(response.headers.get('Retry-After', 60))
    time.sleep(retry_after)
    # Retry request
```

## Further Reading

- Full API Documentation: https://api.example.com/docs
- CLI Documentation: https://example.com/cli
- Human-oriented guides: https://example.com/docs
```

## Appendix B: Implementation Checklist

### Required

- [ ] Create `.well-known/__ai_help` at site root (RFC 8615 compliant)
- [ ] Add required YAML front matter (§3.2.2):
  - [ ] `abt` - Link to dashdash repo
  - [ ] `sub` - Subdirectory files exist
  - [ ] `ver` - Specification version link
  - [ ] `acl` - Access control level
  - [ ] `cli` - CLI alternative URL or `none`
  - [ ] `mcp` - MCP server URL or `none`
  - [ ] `api` - API docs URL or `none`
- [ ] Set appropriate `acl` level (§3.3)
- [ ] Document all required sections (§3.5.1):
  - [ ] Overview
  - [ ] Access Requirements
  - [ ] Site Structure
  - [ ] URL Patterns
  - [ ] Rate Limits and Performance (§3.5.10)
  - [ ] Allowed Operations
  - [ ] Forbidden Operations
  - [ ] Error Handling
- [ ] Include authentication flows (§3.5.5)
- [ ] List allowed and forbidden operations

### Recommended

- [ ] Add quick start example
- [ ] Document date/time formats if applicable (§3.5.3)
- [ ] Provide form field specifications (§3.5.4)
- [ ] Include error handling examples
- [ ] Use visual markers (✅, ❌, ⚠️, 💡)
- [ ] Reference CLI/API/MCP alternatives
- [ ] Add troubleshooting section
- [ ] Include best practices for AI agents
- [ ] Provide input schema specifications (§3.5.6):
  - [ ] Request body schemas
  - [ ] Query parameters
  - [ ] Required headers
  - [ ] Field validation rules
- [ ] Provide output schema specifications (§3.5.7):
  - [ ] Success response structures
  - [ ] Error response format
  - [ ] Field guarantees
  - [ ] Pagination structure
- [ ] Document operational semantics (§3.5.8):
  - [ ] HTTP method idempotency
  - [ ] Idempotency key usage
  - [ ] Side effects by endpoint
  - [ ] Pagination patterns
  - [ ] Partial success handling
  - [ ] CORS behavior
- [ ] Include endpoint capabilities/categories (§3.5.9)
- [ ] Add lifecycle metadata (§3.5.11):
  - [ ] API version information
  - [ ] Deprecated endpoints
  - [ ] Breaking changes by version
  - [ ] Sunset schedules
  - [ ] Migration guides

### Discovery and Integration

- [ ] Add to robots.txt (optional, §3.4.1)
- [ ] Add HTML meta tag (optional, §3.4.2)
- [ ] Add HTTP header (optional, §3.4.3)
- [ ] Add legacy redirect from `/__ai_help.md` (optional, §3.4.4)
- [ ] Create subdirectory files if needed
- [ ] Set `sub: true` in root if hierarchical

### Validation

- [ ] Test with actual AI agent
- [ ] Validate YAML front matter parses correctly
- [ ] Verify all URLs and examples are accurate
- [ ] Review for information disclosure
- [ ] Test rate limit header inclusion
- [ ] Verify error responses match documentation
- [ ] Confirm auth flows work as documented
