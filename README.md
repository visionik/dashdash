# dashdash

**Version:** 0.2.0  
**Status:** Draft  
**Last Updated:** 2026-01-21

## Overview

dashdash is a specification for enabling AI agents to discover and understand how to interact with software tools and services. It provides standardized formats for exposing capabilities, constraints, and usage patterns across four primary access methods: CLI, Web, API, and MCP.

**Primary Goals (in order):**

1. **Enable SKILL.md creation** — Provide structured metadata for creating AI agent skill definitions
2. **Enable effective tool/service usage** — Document how to interact with the tool or service
3. **Enable alternative access discovery** — Cross-reference CLI ↔ API ↔ Web ↔ MCP alternatives

## The Four Access Methods

Modern software services are typically accessible through up to four different interfaces:

| Method | dashdash Spec | Discovery |
|--------|---------------|-----------|
| **CLI** | `--ai-help` flag | Markdown with YAML front matter |
| **Web** | `/llms.txt` file | Extends [llmstxt.org](https://llmstxt.org/) |
| **API** | Via Web or CLI docs | Cross-referenced from other specs |
| **MCP** | `ai_help` method + extensions | JSON-RPC with optional markdown |

## Specifications

### 📋 [CLI `--ai-help` Specification](./proposal-cli--ai-help.md)

Defines the `--ai-help` flag for command-line tools.

```bash
tool --ai-help          # Full help with front matter
tool list --ai-help     # Command-specific help (if subcommand-help: true)
```

**Key Features:**
- YAML front matter with structured metadata
- Eager processing (works without credentials)
- "When to Use" section for skill triggering
- "Quick Reference" section for common operations
- Cross-references to Web, API, and MCP alternatives

### 🌐 [Web llms.txt Enhancements](./proposal--web--llms.txt-enhancements.md)

Extends [llmstxt.org](https://llmstxt.org/) with optional YAML front matter and structured sections.

```
https://example.com/llms.txt          # Root file (per llmstxt.org)
https://example.com/llms-full.txt     # Full inline content (optional)
https://example.com/api/llms.txt      # Subdirectory files (optional)
```

**Key Features:**
- Fully backward compatible with llmstxt.org
- Optional YAML front matter for structured metadata
- "When to Use" and "Quick Reference" sections
- Cross-references to CLI, API, and MCP alternatives
- Access control levels (read, browse, interact, full)

### 🔌 [MCP Server Enhancements](./proposal--mcp--enhancements.md)

Extends MCP servers with discovery metadata and an `ai_help` method.

```json
{
  "method": "ai_help",
  "params": {"format": "markdown"}
}
```

**Key Features:**
- `dashdash` extension in initialize response
- Tool-level metadata (idempotency, side effects, CLI/API equivalents)
- `ai_help` method returning markdown or JSON
- Cross-references to CLI, API, and Web alternatives

### 🔧 [Go/Cobra Integration](./proposal--go--cobra-example.md)

Automate `--ai-help` and MCP server generation for Go CLI tools using Cobra.

```go
// Add to existing Cobra app
dashdash.Integrate(rootCmd, &dashdash.Config{
    Name:        "myapp",
    Description: "CLI for MyApp. Use when user asks to...",
    WebURL:      "https://myapp.com",
    APIURL:      "https://api.myapp.com/docs",
})
```

```bash
myapp --ai-help      # Generated markdown with front matter
myapp --mcp-serve    # Auto-generated MCP server
```

**Key Features:**
- Introspects Cobra command tree automatically
- Generates YAML front matter from config
- Creates "When to Use" from descriptions/annotations
- Builds "Quick Reference" from all commands
- Exposes commands as MCP tools with type mapping
- Annotation system for custom metadata

## Front Matter Structure

All dashdash formats use consistent YAML front matter:

```yaml
---
# Identity (REQUIRED for skill creation)
name: tool-name
description: >
  What the tool does.
  Use when user asks to [specific triggers].

# Specification (REQUIRED)
spec-url: https://github.com/visionik/dashdash
spec-version: 0.2.0

# Access Control (REQUIRED)
access-level: interact    # read | browse | interact | full

# Alternative Access Methods (REQUIRED)
cli-url: https://example.com/cli
api-url: https://api.example.com/docs
web-url: https://example.com/app
mcp-url: https://github.com/example/mcp-server

# Installation (RECOMMENDED)
install:
  brew: org/tap/formula
  npm: "@org/package"
  pip: package-name

# Requirements (RECOMMENDED)
requires:
  bins: [tool]
  os: [darwin, linux]
  auth: [api-key]
  env: [API_KEY]
  scopes: [read:data, write:data]

# Invocation Control (RECOMMENDED)
invocation:
  model-invocable: true
  user-invocable: true

# Documentation (RECOMMENDED)
homepage: https://example.com
repository: https://github.com/org/tool

# Versioning (OPTIONAL)
content-version: 1.0.0
last-updated: 2026-01-21
---
```

## Required Content Sections

All dashdash outputs MUST include:

### When to Use

Explicit triggers for skill creation:

```markdown
## When to Use

Use this tool when the user asks to:
- Manage projects or tasks
- Track team progress
- Generate reports

Do NOT use this tool for:
- Personal todo lists (use local apps)
- File storage (use dedicated services)
```

### Quick Reference

Concise command/operation summary (10-20 items):

```markdown
## Quick Reference

- **List projects:** `tool projects list` / `GET /api/projects`
- **Create project:** `tool projects create --name X` / `POST /api/projects`
- **Get project:** `tool projects get ID` / `GET /api/projects/{id}`
```

## Cross-Platform Discovery

Each access method documents alternatives, enabling agents to choose the best approach:

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│   CLI   │ ←─→ │   Web   │ ←─→ │   API   │ ←─→ │   MCP   │
│--ai-help│     │/llms.txt│     │  /docs  │     │ ai_help │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
     ↑               ↑               ↑               ↑
     └───────────────┴───────────────┴───────────────┘
                    Cross-references
```

## Use Cases

### For Tool/Service Developers

Implement dashdash to help AI agents use your tool correctly:
- Reduce support burden from incorrect AI-generated commands
- Enable automatic SKILL.md generation
- Document rate limits, auth flows, and error handling
- Cross-reference all access methods

### For AI Agent/Skill Developers

Use dashdash formats to discover and understand tools:
- Check for `--ai-help`, `/llms.txt`, or `ai_help` method
- Parse front matter for skill metadata
- Extract "When to Use" for trigger phrases
- Discover alternative access methods

### For SKILL.md Generation

The `name`, `description`, and "When to Use" section can be directly extracted:

```yaml
# From --ai-help output:
name: acme-cli
description: >
  CLI for Acme project management.
  Use when user asks to manage projects or track tasks.
```

```markdown
# Generated SKILL.md:
---
name: acme-cli
description: CLI for Acme project management. Use when user asks to manage projects or track tasks.
---

# acme-cli

## When to Use
[extracted from --ai-help]

## Quick Reference
[extracted from --ai-help]
```

## Relationship to Existing Standards

dashdash complements existing standards:

| Standard | Relationship |
|----------|--------------|
| [llmstxt.org](https://llmstxt.org/) | Web spec extends llms.txt with optional front matter |
| [MCP](https://modelcontextprotocol.io/) | MCP spec adds discovery extensions |
| [OpenAPI](https://www.openapis.org/) | API docs can be cross-referenced |
| [Agent Skills](https://agentskills.io/) | Front matter aligns with skill formats |
| [Claude Code Skills](https://code.claude.com/docs/en/skills) | Compatible metadata fields |
| robots.txt | Complementary (permissions vs. guidance) |

## Implementation Status

**Status:** Draft specification (v0.2.0) seeking feedback

**Reference Implementations:**
- [OuraCLI](https://github.com/visionik/ouracli) — CLI with `--ai-help`

## Version History

**v0.2.0 (2026-01-21):**
- Added skill creation as primary design goal
- Renamed front matter fields (more descriptive, hyphenated)
- Added required `name` and `description` fields
- Added required "When to Use" and "Quick Reference" sections
- Combined web proposals into llms.txt enhancements
- Added MCP server enhancements specification
- Expanded Go/Cobra integration to cover both `--ai-help` and MCP automation
- Added cross-platform discovery emphasis

**v0.1.0 (2026-01-07):**
- Initial draft with CLI and Web specifications

## Contributing

This specification welcomes feedback:

- File issues for ambiguities or missing use cases
- Share implementations to inform evolution
- Test with AI agents and report findings

## License

This specification is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) — dedicated to the public domain.

## Acknowledgments

This specification is informed by:

- [llmstxt.org](https://llmstxt.org/) — Foundation for web AI documentation
- [Model Context Protocol](https://modelcontextprotocol.io/) — MCP specification
- [Agent Skills](https://agentskills.io/) — Open standard for AI skills
- [Claude Code Skills](https://code.claude.com/docs/en/skills) — Skill format analysis
- Real-world AI agent interactions and error patterns
- OuraCLI reference implementation feedback
