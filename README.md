# dashdash

**Version:** 1.0.0  
**Status:** Draft  
**Last Updated:** 2026-01-07

## Overview

dashdash is a specification for enabling AI agents to discover and understand how to interact with software tools and services. It provides standardized formats for exposing capabilities, constraints, and usage patterns across four primary access methods: Web, API, CLI, and MCP.

## The Four Access Methods

Modern software services are typically accessible through up to four different interfaces:

1. **Web** - Browser-based interfaces (HTML/JavaScript applications)
2. **API** - REST, GraphQL, or other programmatic endpoints
3. **CLI** - Command-line interface tools
4. **MCP** - Model Context Protocol servers

### Current State

Each access method has varying levels of AI agent support:

- **APIs** generally have good documentation, but it's often web-based and optimized for human reading
- **MCP** (Model Context Protocol) has native AI discovery built-in through the `tools/list` mechanism, where agents can query available tools and receive structured metadata including names, descriptions, and input schemas
- **Web** interfaces rely on HTML structure and navigation, which can be ambiguous for agents
- **CLIs** provide `--help` text optimized for humans, not machines

### The dashdash Solution

dashdash brings structured, AI-optimized discovery to all four access methods:

- **CLI**: `--ai-help` flag outputs markdown with YAML front matter containing comprehensive usage guidance
- **Web**: `__ai_help.md` files (at root and subdirectories) provide structured site navigation and interaction rules
- **API**: Web-based API documentation can implement `__ai_help.md` to augment existing OpenAPI/Swagger specs
- **MCP**: Already has native tool discovery; dashdash front matter can reference MCP servers as alternatives

## Key Principles

### 1. Structured Metadata (Front Matter)

All dashdash formats use YAML front matter to provide machine-readable metadata:

```yaml
---
abt: https://github.com/visionik/dashdash
sub: false                    # Hierarchical support (commands/subdirs)
ver: [specification version]   # Which spec version is implemented
acl: read                      # Access control level
web: [url or none]             # Web alternative
cli: [url or none]             # CLI alternative  
mcp: [url or none]             # MCP server alternative
api: [url or none]             # API documentation alternative
---
```

### 2. Cross-Reference Alternatives

Each access method documents alternatives:
- CLI help references web interface, API, and MCP alternatives
- Web `__ai_help.md` references CLI tools, APIs, and MCP servers
- This allows agents to choose the most appropriate access method

### 3. AI-Optimized Content

Unlike human-oriented documentation, dashdash formats prioritize:

- **Explicit negative examples** - "Do NOT use X" instead of omitting X
- **Complete format specifications** - All supported date formats, not just common ones
- **Error message mappings** - Actual errors with root causes and fixes
- **Structured constraints** - Rate limits, quotas, permissions
- **Programmatic recommendations** - Prefer `--json` over human-readable formats

### 4. Hierarchical Structure

Both web and CLI support hierarchical help:

- **Web**: Root `__ai_help.md` + subdirectory files (`/app/__ai_help.md`, `/api/__ai_help.md`)
- **CLI**: Global `--ai-help` + command-level help (`tool list --ai-help`, `tool fetch --ai-help`)

The `sub:` field in front matter indicates whether hierarchical help exists.

### 5. Discoverability

Each format includes multiple discovery mechanisms:

- **CLI**: Mentioned in `--help` output, processed eagerly (before auth/config)
- **Web**: Referenced in `robots.txt`, HTML meta tags, HTTP headers, `.well-known/ai-help`

## Specifications

dashdash consists of two complementary specifications:

### 📋 [CLI Specification](./proposal-cli--ai-help.md)

Defines the `--ai-help` flag for command-line tools.

**Key Features:**
- Eager flag processing (works without credentials)
- YAML front matter with metadata
- Command-level help for complex CLIs
- Comprehensive date/time format documentation
- Output format recommendations (prefer `--json`)
- Error troubleshooting guides

**Example:**
```bash
tool --ai-help          # Shows full help with front matter
tool list --ai-help     # Command-specific help
```

### 🌐 [Web Specification](./proposal-web__ai_help.md)

Defines the `__ai_help.md` file convention for websites and web applications.

**Key Features:**
- Hierarchical file structure (root + subdirectories)
- Access control levels (none, read, browse, interact, full)
- Form field specifications with validation rules
- Authentication flow documentation
- Rate limiting and quota information
- HTTP error code mappings

**Example:**
```
https://example.com/__ai_help.md
https://example.com/app/__ai_help.md
https://example.com/api/__ai_help.md
```

## Use Cases

### For CLI Tool Developers

Implement `--ai-help` to help AI agents use your tool correctly:
- Reduce support burden from AI-generated incorrect commands
- Provide explicit date format specifications
- Recommend programmatic output formats
- Document common errors and solutions

### For Web Application Developers

Add `__ai_help.md` to enable AI agents to interact with your site:
- Define what operations are allowed vs. forbidden
- Specify form field formats and validation
- Document rate limits and authentication
- Provide API alternatives for efficiency

### For AI Agent Developers

Use dashdash formats to discover and understand tools:
- Check for `--ai-help` or `__ai_help.md` before attempting interaction
- Parse front matter to discover alternative access methods
- Use structured error guidance to handle failures gracefully
- Respect `acl` (access control level) constraints

## Relationship to Existing Standards

dashdash complements, rather than replaces, existing documentation standards:

- **OpenAPI/Swagger**: API schemas remain authoritative; `__ai_help.md` can augment with AI-specific guidance
- **man pages**: Traditional Unix documentation for humans; `--ai-help` provides AI-oriented supplement
- **MCP tools/list**: MCP's native discovery is already optimal; dashdash references MCP as an alternative
- **robots.txt**: Web crawling policies; `__ai_help.md` focuses on programmatic interaction

## Implementation Status

**Status:** Draft specification seeking feedback

**Reference Implementation:**
- [OuraCLI](https://github.com/visionik/ouracli) - First CLI to implement `--ai-help`

**Specification Versioning:**
- Uses semantic versioning (MAJOR.MINOR.PATCH)
- MAJOR: Incompatible changes to format or behavior
- MINOR: New recommended sections or fields
- PATCH: Clarifications and examples

## Contributing

This specification is in draft status and welcomes feedback:

- File issues for ambiguities or missing use cases
- Share implementations to inform specification evolution
- Propose additional access methods or metadata fields
- Test with various AI agents and report findings

## Future Considerations

Potential future enhancements being considered:

### Machine-Readable Schemas
- JSON-LD structured data
- Formal grammar definitions for inputs
- OpenAPI-like schemas for CLI commands

### Interactive Discovery
- `--ai-help interactive` for Q&A mode
- Structured queries (`--ai-help dates`, `--ai-help auth`)
- Dynamic help based on user permissions

### Multi-Language Support
- Content negotiation via HTTP headers or env vars
- Multiple language files (`__ai_help.en.md`, `__ai_help.es.md`)

## References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for specifications
- [CommonMark](https://commonmark.org/) - Markdown specification
- [YAML](https://yaml.org/) - Front matter format
- [Model Context Protocol](https://modelcontextprotocol.io/) - MCP specification
- [OpenAPI](https://www.openapis.org/) - API specification format

## License

This specification is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) - dedicated to the public domain.

## Acknowledgments

This specification is informed by:
- Real-world AI agent interactions with CLI tools and web services
- Common error patterns in LLM-generated commands
- The Model Context Protocol's tool discovery mechanism
- Existing standards like OpenAPI, man pages, and robots.txt
- Feedback from the OuraCLI implementation
