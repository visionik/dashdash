# CLI AI Help Specification

**Version:** 1.0.0  
**Status:** Draft  
**Last Updated:** 2025-12-30

## 1. Introduction

This document specifies a standard approach for Command-Line Interface (CLI) tools to provide AI-optimized usage guidance through a dedicated `--ai-help` parameter. As Large Language Models (LLMs) and AI agents increasingly interact with CLI tools, specialized documentation format requirements emerge that differ from traditional human-oriented help text.

### 1.1 Key Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## 2. Rationale

### 2.1 Problem Statement

Traditional `--help` output is optimized for human readability with formatting, examples, and concise summaries. AI agents have different needs:

- **Comprehensive date/time format specifications** - AIs commonly make mistakes with date parsing
- **Explicit negative examples** - "Do NOT use X" is clearer than omitting X
- **Error handling guidance** - Common failure modes and solutions
- **Output format recommendations** - Which format is best for programmatic parsing
- **Troubleshooting guides** - Structured error diagnosis
- **Exhaustive option enumeration** - All valid combinations, not just common ones

### 2.2 Benefits

- **Improved AI accuracy** - Reduces hallucination and incorrect command generation
- **Reduced user friction** - AIs can self-serve without iterative trial-and-error
- **Standardization** - Common flag creates predictable discovery pattern
- **Separation of concerns** - Keeps human help concise while providing AI detail

## 3. Specification

### 3.1 Flag Naming

#### 3.1.1 Primary Flag

CLI tools MUST implement the flag `--ai-help`.

**Rationale:** The term "ai-help" is:
- Self-documenting and immediately recognizable
- Clear about audience (AI agents/LLMs)
- Parallel to existing `--help` convention
- Short and memorable

#### 3.1.2 Alternative Names

CLI tools MAY provide aliases such as:
- `--llm-help`
- `--agent-help`
- `--machine-help`

However, `--ai-help` MUST be the primary and documented option.

#### 3.1.3 Deprecated Names

Historical implementations using `--llmhelp` (no hyphen) SHOULD migrate to `--ai-help` to maintain consistency across the ecosystem.

### 3.2 Behavior

#### 3.2.1 Output and Exit

When invoked, the tool MUST:
1. Output the AI-optimized help content to stdout
2. Exit with status code 0
3. Skip all other processing (similar to `--help`)

#### 3.2.2 Eagerness

The `--ai-help` flag SHOULD be processed eagerly, before:
- Credential validation
- API connections
- File system operations
- Other option validation

This allows AIs to learn tool usage without having prerequisite configuration.

### 3.3 Discoverability

#### 3.3.1 Main Help Integration

The CLI tool's primary help text (accessed via `--help` or no arguments) MUST mention the `--ai-help` option prominently. 

RECOMMENDED placement:
```
CLI tool for accessing [service/data].
💡 LLMs/agents: run '[toolname] --ai-help' for detailed usage guidance.
```

#### 3.3.2 Options List

The `--ai-help` flag MUST appear in the options section of standard help output with a description such as:
```
--ai-help    Show comprehensive usage guide for LLMs/agents and exit.
```

#### 3.3.3 No-Argument Behavior

When a user runs the CLI with no arguments or commands, the tool SHOULD display help content that includes mention of `--ai-help`.

### 3.4 Content Requirements

#### 3.4.1 Format

The output MUST be markdown with YAML front matter (following the dashdash format specification).

##### 3.4.1.1 Required Front Matter

The front matter MUST include:

```yaml
---
abt: https://github.com/visionik/dashdash
sub: false
ver: https://github.com/visionik/dashdash/blob/main/README.md#v1.0.0
acl: read
web: https://example.com
mcp: none
api: https://api.example.com/docs
---
```

**Field Definitions:**

- **abt** (REQUIRED): Link to the dashdash GitHub repository
- **sub** (REQUIRED): Boolean indicating whether the CLI supports command-level `--ai-help` (e.g., `cli list --ai-help`, `cli fetch --ai-help`). Set to `false` if only global `--ai-help` is supported.
- **ver** (REQUIRED): Link to the version of this specification the output implements
- **acl** (REQUIRED): AI usage access level - typically `read` for read-only tools, `interact` for tools that modify data, or `full` for unrestricted access
- **web** (REQUIRED): URL to web interface/application alternative, or `none` if no web alternative exists
- **mcp** (REQUIRED): URL to MCP server documentation, or `none` if no MCP alternative exists
- **api** (REQUIRED): URL to API documentation, or `none` if no API alternative exists

##### 3.4.1.2 Markdown Content

Following the front matter, the markdown content MUST include:
- Clear section headers (using `##` and `###`)
- Code blocks with appropriate language tags
- Lists for enumeration
- Emphasis for warnings (e.g., ⚠️, ❌, ✅ emoji)

#### 3.4.2 Command-Level Help

For CLIs with `sub: true` in the front matter, individual commands MAY implement their own `--ai-help`:

```bash
cli --ai-help          # Global help with front matter
cli list --ai-help     # Command-specific help (no front matter)
cli fetch --ai-help    # Command-specific help (no front matter)
```

**Rules for command-level help:**
- Command-level help output MUST NOT include front matter (only global `--ai-help` has front matter)
- Command-level help SHOULD focus on that command's specific options, examples, and edge cases
- Command-level help SHOULD reference the global help for authentication and general setup

#### 3.4.3 Required Sections

The help content MUST include:

1. **Overview** - Brief tool description and capabilities
2. **Setup/Prerequisites** - Authentication, installation, configuration
3. **Command Reference** - Complete list of commands/subcommands
4. **Input Specification** - Detailed parameter/argument formats
5. **Output Formats** - Available formats and recommendations
6. **Examples** - Common use cases with actual commands
7. **Authentication and Prerequisites** (§3.4.10) - Required credentials, scopes, permissions
8. **Rate Limits and Performance** (§3.4.13) - Quotas, timeouts, payload limits

#### 3.4.4 Recommended Sections

The help content SHOULD include:

1. **Troubleshooting Guide** - Common errors with solutions
2. **Negative Examples** - Explicit "do NOT do this" guidance
3. **Format Conversion Guide** - How to transform common query patterns
4. **Best Practices** - LLM-specific recommendations (e.g., "Always use --json")
5. **Quick Reference Card** - Table of common operations
6. **Error Handling** - Expected failure modes
7. **Input Schema** (§3.4.8) - JSON Schema or equivalent for parameters
8. **Output Schema** (§3.4.9) - Expected response structures
9. **Operational Semantics** (§3.4.11) - Idempotency, side effects, pagination
10. **Capabilities and Tags** (§3.4.12) - Command categorization for discovery
11. **Lifecycle Metadata** (§3.4.14) - Deprecations, versions, EOL schedules

#### 3.4.5 Date/Time Handling

If the CLI accepts date or time parameters, the documentation MUST:
- List ALL supported formats with examples
- List common UNSUPPORTED formats with explanations
- Provide conversion examples for ambiguous queries
- Include timezone handling guidance

Example:
```markdown
### ✅ SUPPORTED DATE FORMATS
- `2025-12-30` - ISO 8601 date
- `"7 days"` - Relative range

### ❌ UNSUPPORTED FORMATS
- `2025-12-30 to 2025-12-31` - Do NOT use "to" syntax
  Error: "Invalid date specification"
```

#### 3.4.6 Output Format Guidance

For CLIs with multiple output formats, the documentation MUST:
- List all available formats
- Explicitly recommend the best format for programmatic consumption
- Warn against formats that are difficult to parse

Example:
```markdown
⚠️ **LLM Best Practice: Always Use --json**

✅ RECOMMENDED for programmatic analysis:
    tool fetch --json

❌ NOT RECOMMENDED for parsing:
    tool fetch --tree
```

#### 3.4.7 Error Examples

The documentation SHOULD include:
- Actual error messages the tool produces
- Root cause explanations
- Corrected command examples

Example:
```markdown
### Error: "Got unexpected extra argument"
**Cause:** Multiple arguments without proper quoting

❌ WRONG: tool fetch 2025-01-01 2025-01-31
✅ CORRECT: tool fetch "2025-01-01 30 days"
```

#### 3.4.8 Input Schema

For CLIs with structured inputs, the documentation SHOULD include JSON Schema or equivalent specifications:

**Requirements:**
- Parameter types (string, integer, boolean, array, object)
- Enumerated values for constrained options
- Required vs. optional fields
- Default values
- Validation constraints (min/max, patterns, length)
- Mutually exclusive parameters
- Parameter dependencies

Example:
```markdown
## Input Schema

### Command: `tool create`

**JSON Schema:**
```json
{
  "type": "object",
  "required": ["name", "type"],
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100,
      "pattern": "^[a-zA-Z0-9-_]+$",
      "description": "Project name, alphanumeric with hyphens/underscores"
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
      "description": "Optional metadata tags"
    },
    "priority": {
      "type": "integer",
      "minimum": 1,
      "maximum": 5,
      "default": 3
    }
  }
}
```

**CLI Mapping:**
- `--name NAME` (required)
- `--type {public|private|internal}` (required)
- `--tags TAG1,TAG2,...` (optional, comma-separated)
- `--priority N` (optional, defaults to 3)

**Constraints:**
- `--type private` requires `--owner` parameter
- Cannot specify both `--template` and `--from-scratch`
```

#### 3.4.9 Output Schema

For CLIs with structured outputs, the documentation SHOULD include expected result schemas:

**Requirements:**
- Output structure for each format (JSON, YAML, etc.)
- Field types and descriptions
- Optional vs. guaranteed fields
- Pagination structure if applicable
- Error response format

Example:
```markdown
## Output Schema

### Command: `tool list --json`

**Success Response:**
```json
{
  "items": [
    {
      "id": "string (format: proj_[a-z0-9]+)",
      "name": "string",
      "type": "string (enum: public|private|internal)",
      "status": "string (enum: active|archived|deleted)",
      "created_at": "string (ISO 8601 datetime)",
      "updated_at": "string (ISO 8601 datetime)",
      "owner": {
        "id": "string",
        "name": "string"
      },
      "tags": ["string"],
      "metadata": "object (optional, custom fields)"
    }
  ],
  "pagination": {
    "total": "integer",
    "page": "integer",
    "per_page": "integer",
    "has_more": "boolean",
    "next_cursor": "string (optional)"
  }
}
```

**Field Guarantees:**
- `id`, `name`, `type`, `status`, `created_at` are always present
- `updated_at` may be null for never-modified items
- `owner` only present if user has permission to view
- `metadata` only present if item has custom fields

**Error Response:**
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": "object (optional)"
  }
}
```
```

#### 3.4.10 Authentication and Prerequisites

The documentation MUST clearly specify authentication requirements:

**Requirements:**
- Required authentication methods
- Scopes or permissions needed
- Token types and formats
- Connection state requirements
- Account linking prerequisites

Example:
```markdown
## Authentication

### Required Credentials

**API Key (Required for all commands):**
- Format: `sk_live_[a-zA-Z0-9]{32}` or `sk_test_[a-zA-Z0-9]{32}`
- Obtain from: https://example.com/settings/api
- Provide via: `TOOL_API_KEY` env var or `--api-key` flag

**OAuth Token (Required for `tool admin` commands):**
- Scopes: `admin:read`, `admin:write`
- Obtain via: `tool login --admin`
- Stored in: `~/.config/tool/oauth_token.json`

### Permission Levels

| Command | Required Permission | Notes |
|---------|--------------------|---------|
| `list` | `read` | Basic API key |
| `fetch` | `read` | Basic API key |
| `create` | `write` | API key + verified email |
| `delete` | `admin` | OAuth token with admin scope |
| `export` | `read` + `export` | Requires paid plan |

### Connection State

- **`tool login`**: Establishes OAuth session, stores token locally
- **`tool whoami`**: Verifies current authentication state
- **`tool logout`**: Clears stored credentials

⚠️ **First-Time Setup:**
```bash
# 1. Get API key from dashboard
export TOOL_API_KEY="sk_live_abc123..."

# 2. For admin commands, complete OAuth
tool login --admin

# 3. Verify authentication
tool whoami
```
```

#### 3.4.11 Operational Semantics

The documentation SHOULD specify operational behavior:

**Requirements:**
- Idempotency guarantees
- Side effects and state changes
- Pagination patterns
- Partial failure handling
- Retry behavior
- Transactional boundaries

Example:
```markdown
## Operational Semantics

### Idempotency

**Idempotent Commands:**
- `tool create --idempotency-key KEY`: Safe to retry with same key
- `tool update ID`: Overwrites previous state (repeatable)
- `tool delete ID`: Returns success even if already deleted

**Non-Idempotent Commands:**
- `tool create` (without idempotency key): May create duplicates
- `tool increment ID`: Each call increments counter

### Side Effects

| Command | Side Effects | Reversible? |
|---------|-------------|-------------|
| `list` | None (read-only) | N/A |
| `create` | Creates resource, sends webhook, logs event | Requires manual delete |
| `delete` | Soft-deletes resource, can restore within 30 days | `tool restore ID` |
| `export` | Generates async export job, sends email when ready | Cannot cancel |

### Pagination

**Cursor-based (Recommended):**
```bash
tool list --json --limit 100
# Returns: {"items": [...], "next_cursor": "abc123"}

tool list --json --limit 100 --cursor abc123
# Returns next page
```

**Offset-based (Deprecated):**
```bash
tool list --page 1 --per-page 50  # Pages 1-N
```

⚠️ Cursor-based pagination is stable across concurrent modifications

### Partial Failures

**Batch Operations:**
```bash
tool delete id1 id2 id3 --json
```

Response includes per-item status:
```json
{
  "results": [
    {"id": "id1", "status": "deleted"},
    {"id": "id2", "status": "error", "error": "Not found"},
    {"id": "id3", "status": "deleted"}
  ],
  "summary": {"success": 2, "failed": 1}
}
```

**Exit Codes:**
- `0`: All operations succeeded
- `1`: Some operations failed (check JSON output)
- `2`: All operations failed

### Transactions

- Single-resource commands are atomic
- Batch operations are NOT transactional (partial success possible)
- Use `--atomic` flag for all-or-nothing batch behavior (may fail entirely)
```

#### 3.4.12 Capabilities and Tags

The documentation SHOULD categorize commands for improved discovery:

**Requirements:**
- Functional categories/domains
- Operation types (read/write/admin)
- Resource types affected
- Use case tags

Example:
```markdown
## Command Capabilities

### By Category

**Resource Management:**
- `tool list`, `tool fetch`, `tool create`, `tool update`, `tool delete`
- Tags: `crud`, `resources`, `core`

**Data Export:**
- `tool export`, `tool backup`, `tool download`
- Tags: `export`, `data`, `compliance`

**Administration:**
- `tool admin users`, `tool admin audit`, `tool admin settings`
- Tags: `admin`, `security`, `management`

**Integration:**
- `tool webhook`, `tool api-key`, `tool oauth`
- Tags: `integration`, `automation`, `api`

### By Operation Type

**Read Operations (Safe, Idempotent):**
- `list`, `fetch`, `search`, `whoami`

**Write Operations (Idempotent with key):**
- `create`, `update`, `upsert`

**Destructive Operations (Irreversible or time-limited):**
- `delete`, `purge`, `reset`

**Admin Operations (Require elevated permissions):**
- `admin *`, `configure`, `migrate`
```

#### 3.4.13 Rate Limits and Performance

The documentation MUST specify rate limits and performance characteristics:

**Requirements:**
- Rate limit tiers and quotas
- Timeout values
- Maximum payload sizes
- Batching recommendations
- Retry-After handling
- Caching behavior

Example:
```markdown
## Rate Limits and Performance

### Rate Limits

**By Plan Tier:**
| Tier | Requests/Hour | Burst Limit | Concurrent Requests |
|------|---------------|-------------|--------------------|
| Free | 100 | 10/min | 2 |
| Pro | 1,000 | 100/min | 10 |
| Enterprise | 10,000 | 1,000/min | 50 |

**Rate Limit Headers:**
All responses include:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1736265600  # Unix timestamp
Retry-After: 60  # Seconds (only when rate limited)
```

**Handling Rate Limits:**
```bash
# CLI automatically respects rate limits and retries
tool list --retry-on-rate-limit --max-retries 3

# Check rate limit status
tool rate-limit --json
```

### Timeouts

**Default Timeouts:**
- Connection: 10 seconds
- Read: 30 seconds
- Total request: 60 seconds

**Override Timeouts:**
```bash
tool fetch ID --timeout 120  # 2 minutes for slow operations
```

**Long-Running Operations:**
For exports, migrations, bulk operations:
```bash
tool export --async --json
# Returns: {"job_id": "job_123", "status": "pending"}

tool job status job_123
# Poll until complete
```

### Payload Limits

**Maximum Sizes:**
- Single request body: 10 MB
- File uploads: 100 MB per file
- Batch operations: 1,000 items per request
- JSON response: 50 MB (paginate large datasets)

**Batching Best Practices:**

💡 **Batch small operations:**
```bash
# Good: Single batch request
tool delete id1 id2 id3 ... id100

# Avoid: 100 individual requests
for id in id*; do tool delete $id; done
```

💡 **Use pagination for large lists:**
```bash
# Good: Paginated
tool list --limit 100 --json | jq -r '.next_cursor'

# Avoid: Fetching all at once (may timeout)
tool list --limit 10000
```

### Caching

**Cached Commands:**
- `tool list`: Cached for 60 seconds
- `tool fetch ID`: Cached for 5 minutes

**Cache Control:**
```bash
tool list --no-cache        # Bypass cache
tool list --max-age 300     # Cache up to 5 minutes
tool cache clear            # Clear local cache
```
```

#### 3.4.14 Lifecycle Metadata

The documentation SHOULD include versioning and deprecation information:

**Requirements:**
- Deprecation warnings
- Version ranges and compatibility
- Replacement commands/options
- Breaking change notices
- End-of-life dates

Example:
```markdown
## Lifecycle and Versioning

### Tool Version

**Current Version:** 2.4.0
**Minimum Supported API:** v2.0
**Recommended API:** v2.4

```bash
tool --version  # Check installed version
tool upgrade    # Upgrade to latest
```

### Deprecated Commands

⚠️ **Deprecated in v2.3.0 (Removed in v3.0.0):**

| Deprecated Command | Replacement | Reason |
|-------------------|-------------|--------|
| `tool fetch --format table` | `tool fetch --json \| jq` | Inconsistent formatting |
| `tool list --offset N` | `tool list --cursor TOKEN` | Unstable pagination |
| `tool create --visibility V` | `tool create --type V` | Naming consistency |

**Migration Examples:**

❌ **Old (deprecated):**
```bash
tool create --name "Project" --visibility private
```

✅ **New:**
```bash
tool create --name "Project" --type private
```

### Breaking Changes by Version

**v2.0.0 (Released 2025-01-01):**
- Changed default output from table to JSON
- Removed `--legacy-auth` flag
- Required `--api-key` for all commands

**v1.5.0 → v2.0.0 Migration:**
```bash
# Export data before upgrade
tool-v1 list --all --json > backup.json

# Upgrade tool
tool upgrade --to v2.0.0

# Update authentication
export TOOL_API_KEY="your_key_here"

# Verify migration
tool whoami
```

### End-of-Life Schedule

| Version | EOL Date | Status | Action Required |
|---------|----------|--------|----------------|
| v1.x | 2025-06-01 | ⚠️ Deprecated | Upgrade to v2.x |
| v2.x | 2027-01-01 | ✅ Supported | Current stable |
| v3.x | TBD | 🚧 Beta | Preview available |

**Support Policy:**
- Security fixes: Current major version only
- Bug fixes: Current + previous major version
- New features: Current major version only
```

### 3.5 Content Style

#### 3.5.1 Verbosity

AI help content SHOULD be more verbose than human help. Explicit is better than concise.

#### 3.5.2 Tone

Content SHOULD:
- Be direct and imperative
- Use active voice
- Avoid ambiguity
- Repeat important information if needed

#### 3.5.3 Visual Markers

Content SHOULD use emoji or symbols to highlight:
- ✅ Correct examples
- ❌ Incorrect examples  
- ⚠️ Important warnings
- 💡 Tips and best practices
- 🔧 Troubleshooting steps

## 4. Implementation Guidance

### 4.1 File vs. Inline

Implementations MAY either:
1. Store help content in a separate markdown file (e.g., `docs/ai-help.md`)
2. Embed content inline in source code

RECOMMENDED approach: Separate markdown file for easier maintenance and version control.

### 4.2 Maintenance

The AI help content SHOULD:
- Be updated whenever command syntax changes
- Include version information or date stamp
- Be reviewed by both humans and AIs during testing
- Track common AI mistakes and address them in documentation

### 4.3 Testing

Implementers SHOULD:
- Test that `--ai-help` exits without side effects
- Verify markdown renders correctly in common tools
- Validate that examples are copy-paste executable
- Test with actual AI agents to identify gaps

### 4.4 Examples by Framework

#### 4.4.1 Python (Click)
```python
@click.command()
@click.option('--ai-help', is_flag=True, is_eager=True, 
              help='Show AI/LLM usage guide and exit')
def cli(ai_help):
    if ai_help:
        click.echo(load_ai_help())
        raise SystemExit(0)
```

#### 4.4.2 Python (Typer)
```python
def ai_help_callback(value: bool) -> None:
    if value:
        typer.echo(show_ai_help())
        raise typer.Exit()

@app.callback()
def main(
    ai_help: bool = typer.Option(
        False, "--ai-help",
        callback=ai_help_callback,
        is_eager=True,
        help="Show AI/LLM usage guide"
    )
): ...
```

#### 4.4.3 Go (Cobra)
```go
var aiHelpFlag bool

func init() {
    rootCmd.PersistentFlags().BoolVar(&aiHelpFlag, 
        "ai-help", false, 
        "Show AI/LLM usage guide and exit")
}

func Execute() {
    if aiHelpFlag {
        fmt.Println(getAIHelp())
        os.Exit(0)
    }
    rootCmd.Execute()
}
```

## 5. Evolution and Versioning

### 5.1 Specification Versioning

This specification uses semantic versioning (MAJOR.MINOR.PATCH):
- MAJOR: Incompatible changes to flag name or behavior
- MINOR: New recommended sections or practices
- PATCH: Clarifications and examples

### 5.2 Content Versioning

Individual CLI tools MAY include version information in the front matter. RECOMMENDED format:
```yaml
---
abt: https://github.com/visionik/dashdash
sub: false
ver: https://github.com/visionik/dashdash/blob/main/README.md#v1.0.0
acl: read
web: none
mcp: none
api: none
content_version: 2.1.0
last_updated: 2025-12-30
---
```

## 6. Security Considerations

### 6.1 Information Disclosure

AI help content SHOULD NOT include:
- API keys, tokens, or credentials (even examples)
- Internal system paths beyond necessary examples
- Sensitive business logic or rate limits
- Security vulnerabilities or known exploits

### 6.2 Injection Risks

If AI help content is dynamically generated:
- Input MUST be sanitized
- Template injection MUST be prevented
- Content SHOULD be static when possible

## 7. Future Considerations

### 7.1 Machine-Readable Format

Future versions MAY specify:
- JSON schema for structured help content
- OpenAPI-like command specifications
- Formal grammar definitions for inputs

### 7.2 Interactive Mode

Future versions MAY specify:
- `--ai-help interactive` for Q&A mode
- Structured queries (e.g., `--ai-help dates`)
- Multi-language support

## 8. References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for use in RFCs
- [CommonMark](https://commonmark.org/) - Markdown specification
- [CLI Guidelines](https://clig.dev/) - General CLI best practices
- [GNU Standards for CLIs](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html)

## 9. Acknowledgments

This specification is informed by:
- Real-world LLM interactions with CLI tools
- Common patterns in AI agent error logs
- User feedback on AI-generated command suggestions
- The OuraCLI reference implementation

## Appendix A: Complete Example

### Example Output: `tool --ai-help`

```markdown
---
abt: https://github.com/visionik/dashdash
sub: true
ver: https://github.com/visionik/dashdash/blob/main/README.md#v1.0.0
acl: interact
web: https://example.com/app
mcp: https://github.com/example/tool-mcp
api: https://api.example.com/docs
content_version: 1.2.0
last_updated: 2026-01-07
---

# Tool AI Help Guide

## Overview

Tool is a command-line interface for managing projects and tasks. It supports reading, creating, and updating resources via API.

## Setup/Prerequisites

### Installation

```bash
pip install tool
# or
brew install tool
```

### Authentication

Tool requires an API key:

1. Register at https://example.com/register
2. Generate API key at https://example.com/settings/api
3. Set environment variable:
   ```bash
   export TOOL_API_KEY="your_key_here"
   ```

Alternatively, pass key directly:
```bash
tool --api-key YOUR_KEY list
```

⚠️ **Never commit API keys to version control**

## Command Reference

This CLI supports command-level `--ai-help` for detailed guidance:

- `tool list --ai-help` - List resources
- `tool fetch --ai-help` - Fetch specific resource
- `tool create --ai-help` - Create new resource
- `tool update --ai-help` - Update existing resource

### Global Options

- `--api-key KEY` - API authentication key
- `--json` - Output in JSON format (RECOMMENDED for AI agents)
- `--verbose` - Enable debug logging
- `--version` - Show version and exit
- `--help` - Show human-oriented help
- `--ai-help` - Show this AI-oriented help

## Input Specification

### Date/Time Formats

#### ✅ SUPPORTED FORMATS

- `2026-01-07` - ISO 8601 date
- `2026-01-07T15:30:00Z` - ISO 8601 datetime (UTC)
- `"7 days"` - Relative time range
- `"yesterday"`, `"today"`, `"last week"` - Natural language

#### ❌ UNSUPPORTED FORMATS

- `01/07/2026` - Ambiguous MM/DD/YYYY format
  **Error:** "Invalid date format"
  **Fix:** Use ISO 8601: `2026-01-07`

- `2026-01-07 to 2026-01-14` - "to" syntax not supported
  **Error:** "Got unexpected extra argument"
  **Fix:** Use quoted range: `"2026-01-07 7 days"`

### Resource IDs

- Format: `proj_` prefix + alphanumeric (e.g., `proj_abc123`)
- Case-sensitive
- Obtain from `tool list` command

## Output Formats

⚠️ **LLM Best Practice: Always Use --json**

✅ RECOMMENDED for programmatic parsing:
```bash
tool list --json
```

Output structure:
```json
{
  "items": [
    {
      "id": "proj_123",
      "name": "Project Name",
      "status": "active",
      "created_at": "2026-01-01T00:00:00Z"
    }
  ],
  "total": 1
}
```

❌ NOT RECOMMENDED for parsing:
```bash
tool list --tree  # Visual tree format, hard to parse
tool list         # Human-readable table, formatting may change
```

## Examples

### List All Projects

```bash
tool list --json
```

### Fetch Specific Project

```bash
tool fetch proj_abc123 --json
```

### Create New Project

```bash
tool create --name "New Project" --status active --json
```

### Filter by Date Range

```bash
tool list --after 2026-01-01 --before 2026-01-31 --json
```

## Troubleshooting

### Error: "API key not found"

**Cause:** Missing or invalid API key

**Solution:**
1. Check environment variable: `echo $TOOL_API_KEY`
2. Verify key at https://example.com/settings/api
3. Pass key explicitly: `tool --api-key YOUR_KEY list`

### Error: "Invalid date format"

**Cause:** Non-ISO 8601 date format used

❌ WRONG:
```bash
tool list --after 01/07/2026
```

✅ CORRECT:
```bash
tool list --after 2026-01-07
```

### Error: "Got unexpected extra argument"

**Cause:** Unquoted multi-word arguments or date ranges

❌ WRONG:
```bash
tool create --name My Project
tool list --after 2026-01-01 7 days
```

✅ CORRECT:
```bash
tool create --name "My Project"
tool list --after "2026-01-01 7 days"
```

## Best Practices

💡 **Always use --json flag** for reliable parsing

💡 **Store API key in environment variable** rather than passing inline

💡 **Use ISO 8601 dates** to avoid ambiguity

💡 **Check command exit codes**:
- `0` - Success
- `1` - General error
- `2` - Authentication error
- `3` - Invalid arguments

💡 **Handle rate limits gracefully** - API allows 1000 requests/hour

## Alternative Access Methods

### Web Interface

Access the same functionality via web app:
- URL: https://example.com/app
- Supports all CLI operations through GUI

### MCP Server

Use the Model Context Protocol server for AI agent integration:
- Repository: https://github.com/example/tool-mcp
- Provides structured API for LLM interactions

### REST API

Direct API access for custom integrations:
- Documentation: https://api.example.com/docs
- OpenAPI spec: https://api.example.com/openapi.json

## Quick Reference

| Task | Command |
|------|--------|
| List all | `tool list --json` |
| Get one | `tool fetch RESOURCE_ID --json` |
| Create | `tool create --name "Name" --json` |
| Update | `tool update RESOURCE_ID --status active --json` |
| Filter by date | `tool list --after 2026-01-01 --json` |
| Help for command | `tool COMMAND --ai-help` |
```

### Reference Implementation

See OuraCLI's implementation:
- Flag definition: `src/ouracli/cli.py`
- Help content: `src/ouracli/llm_help.py`
- User-facing result: `ouracli --ai-help`

Key features demonstrated:
- YAML front matter with all required fields
- Prominent discoverability in main help
- Comprehensive date format documentation with negative examples
- Output format recommendations for AI parsing
- Troubleshooting guide with actual error messages
- Quick reference card for common operations
- Explicit best practices for LLMs
- Alternative access method links (web, MCP, API)

## Appendix B: Implementation Checklist

### Required

- [ ] Add `--ai-help` flag to CLI
- [ ] Make flag eager (processes before other validation)
- [ ] Output markdown with YAML front matter to stdout
- [ ] Exit with code 0 after output
- [ ] Include required front matter fields (§3.4.1.1):
  - [ ] `abt` - Link to dashdash repo
  - [ ] `sub` - Command-level help support
  - [ ] `ver` - Specification version link
  - [ ] `acl` - Access control level
  - [ ] `web` - Web alternative URL or `none`
  - [ ] `mcp` - MCP server URL or `none`
  - [ ] `api` - API docs URL or `none`
- [ ] Include all required sections (§3.4.3):
  - [ ] Overview
  - [ ] Setup/Prerequisites
  - [ ] Command Reference
  - [ ] Input Specification
  - [ ] Output Formats
  - [ ] Examples
  - [ ] Authentication and Prerequisites (§3.4.10)
  - [ ] Rate Limits and Performance (§3.4.13)
- [ ] Mention `--ai-help` in main help text
- [ ] Add `--ai-help` to options list

### Recommended

- [ ] Document all date/time formats if applicable (§3.4.5)
- [ ] Provide output format recommendations (§3.4.6)
- [ ] Include error examples with solutions (§3.4.7)
- [ ] Add troubleshooting section
- [ ] Include best practices for AI agents
- [ ] Use visual markers (✅, ❌, ⚠️, 💡)
- [ ] Add quick reference table
- [ ] Document alternative access methods
- [ ] Include command-level `--ai-help` if `sub: true` (§3.4.2)
- [ ] Provide input schema specifications (§3.4.8)
- [ ] Provide output schema specifications (§3.4.9)
- [ ] Document operational semantics (§3.4.11):
  - [ ] Idempotency guarantees
  - [ ] Side effects
  - [ ] Pagination patterns
  - [ ] Partial failure handling
- [ ] Include command capabilities/tags (§3.4.12)
- [ ] Add lifecycle metadata (§3.4.14):
  - [ ] Deprecation warnings
  - [ ] Version compatibility
  - [ ] EOL schedules

### Validation

- [ ] Test with actual AI agent
- [ ] Validate YAML front matter parses correctly
- [ ] Verify examples are copy-paste executable
- [ ] Review for information disclosure
- [ ] Test that flag exits without side effects
- [ ] Verify markdown renders correctly
