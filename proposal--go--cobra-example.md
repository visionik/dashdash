# Go/Cobra dashdash Integration

**Version:** 0.2.0  
**Status:** Draft  
**Last Updated:** 2026-01-21

## Overview

This document describes how to use [Cobra](https://github.com/spf13/cobra) to automatically generate:

1. **`--ai-help` output** — Markdown with YAML front matter (per CLI spec)
2. **MCP server** — Auto-generated from command tree
3. **Cross-references** — Link between CLI, MCP, API, and Web access methods

The goal is **minimal code changes** for existing Cobra CLIs to become fully dashdash-compliant.

## Architecture

```
┌─────────────────────┐
│   Cobra CLI App     │
│     (existing)      │
└──────────┬──────────┘
           │
           ├─→ --ai-help     → Markdown + YAML front matter
           │
           ├─→ --mcp-serve   → MCP server (JSON-RPC over stdio)
           │
           └─→ Normal Mode   → Standard CLI execution
                   
           ┌────────────────────────────────────┐
           │         cobra-dashdash             │
           │  ┌─────────────┐ ┌──────────────┐  │
           │  │ AI Help Gen │ │  MCP Bridge  │  │
           │  │ - Introspect│ │ - Introspect │  │
           │  │ - Format MD │ │ - JSON-RPC   │  │
           │  │ - YAML FM   │ │ - Execute    │  │
           │  └─────────────┘ └──────────────┘  │
           └────────────────────────────────────┘
```

## Quick Start

### Basic Integration

```go
package main

import (
    "os"
    "github.com/spf13/cobra"
    "github.com/visionik/cobra-dashdash"
)

func main() {
    rootCmd := &cobra.Command{
        Use:   "myapp",
        Short: "My CLI application",
        Long:  "A longer description for my CLI application.",
    }
    
    // Add your commands
    rootCmd.AddCommand(searchCmd, listCmd, createCmd)
    
    // Add dashdash integration (adds --ai-help and --mcp-serve flags)
    dashdash.Integrate(rootCmd, &dashdash.Config{
        Name:        "myapp",
        Description: "CLI for MyApp. Use when user asks to search, list, or create items.",
        Version:     "1.0.0",
        
        // Alternative access methods
        WebURL: "https://myapp.com",
        APIURL: "https://api.myapp.com/docs",
        MCPURL: "https://github.com/myorg/myapp-mcp",
        
        // Installation
        Install: map[string]string{
            "brew": "myorg/tap/myapp",
            "go":   "github.com/myorg/myapp@latest",
        },
        
        // Requirements
        Requires: &dashdash.Requirements{
            Auth: []string{"api-key"},
            Env:  []string{"MYAPP_API_KEY"},
        },
    })
    
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}
```

### Usage

```bash
# Normal CLI
myapp search "query"

# AI help (full markdown with front matter)
myapp --ai-help

# Command-specific help
myapp search --ai-help

# MCP server mode
myapp --mcp-serve
```

## Part 1: Automatic `--ai-help` Generation

### How It Works

The `--ai-help` flag introspects the Cobra command tree and generates:

1. **YAML front matter** with dashdash metadata
2. **When to Use** section from command descriptions
3. **Quick Reference** from all leaf commands
4. **Command Reference** with full details
5. **Cross-references** to alternative access methods

### Generated Output Example

```bash
myapp --ai-help
```

Produces:

```markdown
---
name: myapp
description: >
  CLI for MyApp.
  Use when user asks to search, list, or create items.

spec-url: https://github.com/visionik/dashdash
spec-version: 0.2.0

subcommand-help: true
access-level: interact

cli-url: none
api-url: https://api.myapp.com/docs
web-url: https://myapp.com
mcp-url: https://github.com/myorg/myapp-mcp

install:
  brew: myorg/tap/myapp
  go: github.com/myorg/myapp@latest

requires:
  auth:
    - api-key
  env:
    - MYAPP_API_KEY

invocation:
  model-invocable: true
  user-invocable: true

homepage: https://myapp.com
content-version: 1.0.0
---

# myapp

> CLI for MyApp

## When to Use

Use this tool when the user asks to:
- Search for items in MyApp
- List available resources
- Create new items

## Quick Reference

- **Search items:** `myapp search [query] --limit 10`
- **List all:** `myapp list --format json`
- **Create item:** `myapp create --name "Name" --type default`

## Command Reference

### myapp search

Search for items.

**Usage:** `myapp search [query] [flags]`

**Arguments:**
- `query` (required): Search query string

**Flags:**
- `--limit, -l` (int): Maximum results (default: 10)
- `--format, -f` (string): Output format (default: "table")

**Example:**
```bash
myapp search "golang" --limit 5 --format json
```

### myapp list

List all items.

**Usage:** `myapp list [flags]`

**Flags:**
- `--format, -f` (string): Output format (default: "table")
- `--all, -a` (bool): Include archived items

### myapp create

Create a new item.

**Usage:** `myapp create [flags]`

**Flags:**
- `--name, -n` (string, required): Item name
- `--type, -t` (string): Item type (default: "default")

## Authentication

Set the `MYAPP_API_KEY` environment variable:

```bash
export MYAPP_API_KEY="your_api_key"
```

## Alternative Access Methods

- **Web:** https://myapp.com
- **API:** https://api.myapp.com/docs
- **MCP:** https://github.com/myorg/myapp-mcp
```

### Command-Level Help

With `subcommand-help: true`, individual commands also support `--ai-help`:

```bash
myapp search --ai-help
```

Produces focused documentation for just that command (no front matter).

### Customizing AI Help

#### Add "When to Use" Triggers

Use annotations to specify trigger phrases:

```go
searchCmd := &cobra.Command{
    Use:   "search [query]",
    Short: "Search for items",
    Annotations: map[string]string{
        "dashdash:triggers": "search items, find items, look up items",
    },
}
```

#### Add Examples

```go
searchCmd := &cobra.Command{
    Use:     "search [query]",
    Short:   "Search for items",
    Example: `  myapp search "golang" --limit 5
  myapp search "api" --format json`,
}
```

#### Mark Commands as Dangerous

```go
deleteCmd := &cobra.Command{
    Use:   "delete [id]",
    Short: "Delete an item",
    Annotations: map[string]string{
        "dashdash:dangerous":  "true",
        "dashdash:reversible": "false",
    },
}
```

#### Specify Output Schema

```go
listCmd := &cobra.Command{
    Use:   "list",
    Short: "List items",
    Annotations: map[string]string{
        "dashdash:output-format": "json",
        "dashdash:output-schema": `{"type":"array","items":{"type":"object","properties":{"id":{"type":"string"},"name":{"type":"string"}}}}`,
    },
}
```

## Part 2: Automatic MCP Server Generation

### How It Works

The `--mcp-serve` flag starts an MCP server that:

1. **Introspects** the Cobra command tree
2. **Exposes** each leaf command as an MCP tool
3. **Executes** commands when tools are called
4. **Returns** structured output

### MCP Server Mode

```bash
# Start MCP server (stdio JSON-RPC)
myapp --mcp-serve

# Or with environment variable
MCP_MODE=true myapp
```

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "myapp": {
      "command": "myapp",
      "args": ["--mcp-serve"],
      "env": {
        "MYAPP_API_KEY": "your_api_key"
      }
    }
  }
}
```

### Generated MCP Tools

Each Cobra command becomes an MCP tool:

**Cobra Command:**
```go
searchCmd := &cobra.Command{
    Use:   "search [query]",
    Short: "Search for items",
    Args:  cobra.ExactArgs(1),
}
searchCmd.Flags().IntP("limit", "l", 10, "Maximum results")
searchCmd.Flags().StringP("format", "f", "json", "Output format")
```

**Generated MCP Tool:**
```json
{
  "name": "search",
  "description": "Search for items",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Search query"
      },
      "limit": {
        "type": "integer",
        "description": "Maximum results",
        "default": 10
      },
      "format": {
        "type": "string",
        "description": "Output format",
        "default": "json"
      }
    },
    "required": ["query"]
  }
}
```

### Nested Commands

Nested commands are flattened with underscores:

```go
// git → remote → add
gitCmd := &cobra.Command{Use: "git"}
remoteCmd := &cobra.Command{Use: "remote"}
addCmd := &cobra.Command{Use: "add [name] [url]", Short: "Add remote"}

remoteCmd.AddCommand(addCmd)
gitCmd.AddCommand(remoteCmd)
```

Becomes MCP tool: `git_remote_add`

### Enhanced MCP Metadata

The MCP server includes dashdash extensions:

```json
{
  "protocolVersion": "2024-11-05",
  "serverInfo": {
    "name": "myapp",
    "version": "1.0.0"
  },
  "capabilities": {
    "tools": {}
  },
  "dashdash": {
    "specVersion": "0.2.0",
    "identity": {
      "name": "myapp",
      "description": "CLI for MyApp. Use when user asks to search, list, or create items."
    },
    "accessLevel": "interact",
    "alternativeAccess": {
      "cliUrl": null,
      "apiUrl": "https://api.myapp.com/docs",
      "webUrl": "https://myapp.com"
    }
  }
}
```

### Tool-Level Extensions

Each tool includes operational metadata:

```json
{
  "name": "create",
  "description": "Create a new item",
  "inputSchema": {...},
  "dashdash": {
    "category": "write",
    "operationType": "write",
    "idempotent": false,
    "sideEffects": ["creates resource"],
    "reversible": true,
    "reverseMethod": "delete",
    "cliEquivalent": "myapp create --name {name}",
    "apiEquivalent": "POST /api/items"
  }
}
```

### `ai_help` Method

The MCP server also exposes an `ai_help` method:

```json
{
  "method": "ai_help",
  "params": {"format": "markdown"}
}
```

Returns the same content as `--ai-help`.

## Part 3: Type Mapping

### Cobra to JSON Schema

| Cobra Type | JSON Schema | Notes |
|------------|-------------|-------|
| `string` | `string` | Direct |
| `int`, `int64` | `integer` | Direct |
| `float64` | `number` | Direct |
| `bool` | `boolean` | Direct |
| `stringSlice` | `array` of `string` | Comma-separated |
| `intSlice` | `array` of `integer` | Comma-separated |
| `stringToString` | `object` | key=value pairs |
| `duration` | `string` with pattern | e.g., "5m", "30s" |
| `count` | `integer` | Incremental (-v -v -v) |

### Argument Extraction

Arguments are extracted from the `Use` string:

```go
Use: "search [query]"           // → optional "query" argument
Use: "delete <id>"              // → required "id" argument  
Use: "echo [message...]"        // → variadic "message" argument
```

Or explicitly annotated:

```go
cmd.Annotations = map[string]string{
    "dashdash:args": `[
        {"name": "query", "type": "string", "required": true, "description": "Search query"}
    ]`,
}
```

## Part 4: Configuration

### Full Configuration

```go
dashdash.Integrate(rootCmd, &dashdash.Config{
    // Identity (required)
    Name:        "myapp",
    Description: "CLI for MyApp. Use when user asks to...",
    Version:     "1.0.0",
    
    // Access level
    AccessLevel: "interact", // read | browse | interact | full
    
    // Alternative access methods
    WebURL: "https://myapp.com",
    APIURL: "https://api.myapp.com/docs",
    MCPURL: "https://github.com/myorg/myapp-mcp",
    
    // Installation
    Install: map[string]string{
        "brew": "myorg/tap/myapp",
        "go":   "github.com/myorg/myapp@latest",
        "npm":  "@myorg/myapp",
    },
    
    // Requirements
    Requires: &dashdash.Requirements{
        Bins:   []string{"myapp"},
        OS:     []string{"darwin", "linux"},
        Auth:   []string{"api-key"},
        Env:    []string{"MYAPP_API_KEY"},
        Scopes: []string{"read:items", "write:items"},
    },
    
    // Invocation control
    Invocation: &dashdash.Invocation{
        ModelInvocable: true,
        UserInvocable:  true,
    },
    
    // Documentation
    Homepage:   "https://myapp.com",
    Repository: "https://github.com/myorg/myapp",
    StatusPage: "https://status.myapp.com",
    
    // Rate limits
    RateLimit: "1000/hour",
    
    // MCP options
    MCPOptions: &dashdash.MCPOptions{
        // Filter which commands to expose
        CommandFilter: func(cmd *cobra.Command) bool {
            return !cmd.Hidden && cmd.Runnable()
        },
        
        // Force JSON output in MCP mode
        ForceJSONOutput: true,
        
        // Parse output
        OutputParser: func(raw string) (interface{}, error) {
            var result interface{}
            json.Unmarshal([]byte(raw), &result)
            return result, nil
        },
    },
    
    // AI help options
    AIHelpOptions: &dashdash.AIHelpOptions{
        // Include examples in quick reference
        IncludeExamples: true,
        
        // Group commands by category
        GroupByCategory: true,
        
        // Custom sections
        CustomSections: map[string]string{
            "Rate Limits": "Free: 100/hour, Pro: 1000/hour",
        },
    },
})
```

### Environment Variables

```bash
# Force MCP mode
MCP_MODE=true myapp

# Disable AI help flag
DASHDASH_DISABLE_AI_HELP=true myapp

# Disable MCP serve flag  
DASHDASH_DISABLE_MCP_SERVE=true myapp
```

## Part 5: Advanced Features

### Custom Triggers from Annotations

```go
rootCmd.Annotations = map[string]string{
    "dashdash:when-to-use": `
- Search for items by keyword
- List all available resources
- Create new items with metadata
- Delete items by ID`,
    "dashdash:when-not-to-use": `
- File system operations (use filesystem tools)
- Code editing (use code editor)`,
}
```

### API Equivalents

Document API equivalents for each command:

```go
createCmd.Annotations = map[string]string{
    "dashdash:api-equivalent": "POST /api/items",
    "dashdash:api-body":       `{"name": "{name}", "type": "{type}"}`,
}
```

### Idempotency Markers

```go
updateCmd.Annotations = map[string]string{
    "dashdash:idempotent":     "true",
    "dashdash:side-effects":   "updates resource, sends webhook",
    "dashdash:reversible":     "true",
    "dashdash:reverse-method": "restore",
}
```

### Error Documentation

```go
searchCmd.Annotations = map[string]string{
    "dashdash:errors": `[
        {"code": "AUTH_FAILED", "message": "Invalid API key", "resolution": "Check MYAPP_API_KEY"},
        {"code": "RATE_LIMITED", "message": "Too many requests", "resolution": "Wait and retry"}
    ]`,
}
```

## Implementation

### Core Functions

```go
// Integrate adds --ai-help and --mcp-serve to a Cobra command
func Integrate(rootCmd *cobra.Command, config *Config)

// GenerateAIHelp produces markdown with YAML front matter
func GenerateAIHelp(rootCmd *cobra.Command, config *Config) string

// ServeMCP starts an MCP server over stdio
func ServeMCP(rootCmd *cobra.Command, config *Config) error

// CommandToMCPTool converts a Cobra command to MCP tool definition
func CommandToMCPTool(cmd *cobra.Command) MCPTool
```

### Types

```go
type Config struct {
    Name        string
    Description string
    Version     string
    AccessLevel string
    
    WebURL string
    APIURL string
    MCPURL string
    
    Install  map[string]string
    Requires *Requirements
    
    Invocation *Invocation
    
    Homepage   string
    Repository string
    StatusPage string
    RateLimit  string
    
    MCPOptions    *MCPOptions
    AIHelpOptions *AIHelpOptions
}

type Requirements struct {
    Bins   []string
    OS     []string
    Auth   []string
    Env    []string
    Scopes []string
}

type Invocation struct {
    ModelInvocable bool
    UserInvocable  bool
    AllowedTools   []string
}

type MCPTool struct {
    Name        string     `json:"name"`
    Description string     `json:"description"`
    InputSchema JSONSchema `json:"inputSchema"`
    Dashdash    *ToolMeta  `json:"dashdash,omitempty"`
}

type ToolMeta struct {
    Category      string   `json:"category,omitempty"`
    OperationType string   `json:"operationType,omitempty"`
    Idempotent    bool     `json:"idempotent,omitempty"`
    SideEffects   []string `json:"sideEffects,omitempty"`
    CLIEquivalent string   `json:"cliEquivalent,omitempty"`
    APIEquivalent string   `json:"apiEquivalent,omitempty"`
}
```

## Checklist

### For `--ai-help`

- [ ] YAML front matter with all required fields
- [ ] "When to Use" section
- [ ] "Quick Reference" section
- [ ] Full command reference
- [ ] Cross-references to API, Web, MCP
- [ ] Subcommand-level `--ai-help` support

### For MCP Server

- [ ] `tools/list` with all commands
- [ ] `tools/call` execution
- [ ] `ai_help` method
- [ ] dashdash extensions in server info
- [ ] Tool-level operational metadata
- [ ] JSON output forcing

### Annotations Supported

- `dashdash:triggers` — Trigger phrases for When to Use
- `dashdash:when-to-use` — Full When to Use content
- `dashdash:when-not-to-use` — Negative examples
- `dashdash:api-equivalent` — REST API equivalent
- `dashdash:idempotent` — Idempotency marker
- `dashdash:side-effects` — Side effect list
- `dashdash:reversible` — Can be undone
- `dashdash:reverse-method` — Undo command
- `dashdash:dangerous` — Requires confirmation
- `dashdash:output-format` — Expected output format
- `dashdash:output-schema` — JSON schema for output
- `dashdash:errors` — Common errors with resolutions
- `dashdash:args` — Explicit argument specifications

## References

- [Cobra Documentation](https://github.com/spf13/cobra)
- [dashdash CLI --ai-help Spec](./proposal-cli--ai-help.md)
- [dashdash MCP Enhancements](./proposal--mcp--enhancements.md)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [JSON Schema](https://json-schema.org/)

## License

This specification is released under [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).
