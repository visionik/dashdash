# Cobra to MCP Bridge Specification

**Version:** 0.1.0-draft  
**Status:** Proposal  
**Last Updated:** 2025-12-31

## Overview

This specification defines a standard approach for automatically converting [Cobra](https://github.com/spf13/cobra) CLI applications into [Model Context Protocol (MCP)](https://modelcontextprotocol.io) servers, enabling AI assistants to interact with command-line tools through a structured, introspectable interface.

## Goals

- **Zero-modification conversion**: Existing Cobra CLIs should become MCP servers with minimal code changes
- **Full metadata preservation**: Command descriptions, flags, arguments, and help text should map to MCP tool schemas
- **Bidirectional execution**: Convert MCP tool calls to CLI invocations and CLI output back to structured MCP responses
- **Type safety**: Maintain type information across the CLI ↔ MCP boundary

## Architecture

```
┌─────────────────┐
│  Cobra CLI App  │
│   (existing)    │
└────────┬────────┘
         │
         ├─→ Normal Mode: Execute as CLI
         │
         └─→ MCP Mode: Introspect & Serve
                 │
                 ↓
         ┌───────────────┐
         │ MCP Bridge    │
         │ - Introspect  │
         │ - Transform   │
         │ - Execute     │
         └───────┬───────┘
                 │
                 ↓
         ┌───────────────┐
         │  JSON-RPC     │
         │  (stdio)      │
         └───────────────┘
```

## Core Components

### 1. Command Introspection

Walk the Cobra command tree and extract metadata:

```go
type MCPTool struct {
    Name        string     `json:"name"`
    Description string     `json:"description"`
    InputSchema JSONSchema `json:"inputSchema"`
    OutputSchema *JSONSchema `json:"outputSchema,omitempty"`
}

func CommandToMCPTool(cmd *cobra.Command) MCPTool {
    tool := MCPTool{
        Name:        getFullCommandName(cmd),
        Description: getDescription(cmd),
        InputSchema: JSONSchema{
            Type:       "object",
            Properties: make(map[string]Property),
            Required:   []string{},
        },
    }
    
    // Extract flags
    extractFlags(cmd, &tool.InputSchema)
    
    // Extract positional arguments
    extractArguments(cmd, &tool.InputSchema)
    
    return tool
}
```

### 2. Type Mapping

Map Cobra/pflag types to JSON Schema types:

| Cobra Type | JSON Schema Type | Notes |
|------------|------------------|-------|
| `string` | `string` | Direct mapping |
| `int`, `int32`, `int64`, `uint` | `integer` | Direct mapping |
| `float32`, `float64` | `number` | Direct mapping |
| `bool` | `boolean` | Direct mapping |
| `stringSlice` | `array` with `items: {type: "string"}` | Comma-separated in CLI |
| `intSlice` | `array` with `items: {type: "integer"}` | Comma-separated in CLI |
| `stringToString` | `object` with `additionalProperties: {type: "string"}` | key=value pairs |
| `duration` | `string` with pattern | e.g., "5m", "30s" |
| `IP`, `IPNet` | `string` with format | format: "ipv4" or "ipv6" |

### 3. Execution Bridge

Convert MCP tool calls to CLI execution:

```go
func ExecuteMCPTool(toolName string, arguments map[string]interface{}) (*MCPResult, error) {
    // 1. Find the command
    cmd := findCommand(rootCmd, toolName)
    if cmd == nil {
        return nil, fmt.Errorf("tool not found: %s", toolName)
    }
    
    // 2. Build CLI arguments
    cliArgs, err := buildCLIArgs(cmd, arguments)
    if err != nil {
        return nil, err
    }
    
    // 3. Execute command
    output, err := executeCommand(cmd, cliArgs)
    if err != nil {
        return nil, err
    }
    
    // 4. Structure output
    structured := structureOutput(output, cmd)
    
    return &MCPResult{
        Content: []Content{{
            Type: "text",
            Text: structured,
        }},
    }, nil
}
```

### 4. MCP Server

Implement the MCP protocol over stdio:

```go
func ServeMCP(rootCmd *cobra.Command, opts MCPOptions) error {
    server := jsonrpc.NewServer()
    
    // tools/list: Return all available tools
    server.Handle("tools/list", func(ctx context.Context) (ListToolsResult, error) {
        tools := []MCPTool{}
        walkCommands(rootCmd, func(cmd *cobra.Command) {
            if isLeafCommand(cmd) || cmd.Runnable() {
                tools = append(tools, CommandToMCPTool(cmd))
            }
        })
        return ListToolsResult{Tools: tools}, nil
    })
    
    // tools/call: Execute a tool
    server.Handle("tools/call", func(ctx context.Context, req CallToolRequest) (CallToolResult, error) {
        return ExecuteMCPTool(req.Params.Name, req.Params.Arguments)
    })
    
    return server.ServeStdio()
}
```

## Key Challenges & Solutions

### Challenge 1: Positional Arguments

**Problem:** Cobra's argument validators are function pointers, not declarative metadata.

```go
cmd := &cobra.Command{
    Args: cobra.MinimumNArgs(1), // Opaque function!
}
```

**Solutions:**

#### Option A: Parse `Use` String (Simple)
```go
// Parse: "search [query] [path]" → ["query", "path"]
// Parse: "run <file>" → ["file"] (required via <brackets>)
// Parse: "echo [message...]" → ["message"] (variadic via ...)

func parseUsageArgs(use string) []ArgSpec {
    parts := strings.Fields(use)[1:] // Skip command name
    var specs []ArgSpec
    for _, part := range parts {
        required := strings.HasPrefix(part, "<")
        variadic := strings.HasSuffix(part, "...")
        name := strings.Trim(part, "<>[].")
        specs = append(specs, ArgSpec{
            Name:     name,
            Required: required,
            Variadic: variadic,
        })
    }
    return specs
}
```

**Limitations:** Convention-based, no type information, no descriptions.

#### Option B: Annotation System (Recommended)

Extend Cobra commands with explicit metadata:

```go
type ArgSpec struct {
    Name        string  `json:"name"`
    Description string  `json:"description"`
    Type        string  `json:"type"` // "string", "int", "path"
    Required    bool    `json:"required"`
    Variadic    bool    `json:"variadic"`
}

// Attach to command (requires wrapper or Cobra fork)
cmd := &cobra.Command{
    Use:  "search [query] [path]",
    Args: cobra.MinimumNArgs(1),
    // NEW FIELD:
    ArgSpecs: []ArgSpec{
        {Name: "query", Description: "Search query", Type: "string", Required: true},
        {Name: "path", Description: "Search path", Type: "string", Required: false},
    },
}
```

**Implementation:** Use a wrapper struct or custom command type:

```go
type MCPCommand struct {
    *cobra.Command
    ArgSpecs []ArgSpec
}
```

#### Option C: Heuristic Inference (Fallback)

Test validators with different argument counts:

```go
func inferArgRequirements(validator cobra.PositionalArgs) (min, max int) {
    for i := 0; i <= 10; i++ {
        testArgs := make([]string, i)
        err := validator(nil, testArgs)
        if err == nil {
            if min == 0 {
                min = i
            }
            max = i
        }
    }
    return min, max
}
```

**Limitations:** Brittle, slow, no semantic information.

### Challenge 2: Nested Commands

**Problem:** Cobra supports deep hierarchies (`git remote add`), MCP tools are flat.

**Solution:** Flatten with separator and preserve context in description.

```go
func getToolName(cmd *cobra.Command) string {
    parts := []string{}
    for c := cmd; c != nil && c.Name() != ""; c = c.Parent() {
        parts = append([]string{c.Name()}, parts...)
    }
    return strings.Join(parts, "_")
}

// Examples:
// git → remote → add  becomes "git_remote_add"
// kubectl → get → pods becomes "kubectl_get_pods"
```

**Enhancements:**

- Include parent context in description: `"git remote add: Add a new remote repository"`
- Only expose leaf commands (skip intermediate parent-only commands)
- Handle name collisions with namespacing

### Challenge 3: Interactive Prompts

**Problem:** CLIs often use interactive prompts (stdin TTY), but MCP operates over stdio JSON-RPC.

**Solutions:**

#### A. Require Explicit Parameters

Force all interactive inputs to become explicit flags:

```go
// CLI Mode (interactive):
// $ myapp delete file.txt
// Are you sure? (y/n): _

// MCP Mode (explicit):
// Tool call: {"name": "delete", "arguments": {"file": "file.txt", "confirm": true}}
```

Schema definition:
```json
{
  "name": "delete",
  "inputSchema": {
    "type": "object",
    "properties": {
      "file": {"type": "string"},
      "confirm": {"type": "boolean", "description": "Confirm deletion"}
    },
    "required": ["file", "confirm"]
  }
}
```

#### B. Auto-Accept Mode

Use environment variable to bypass prompts:

```go
func init() {
    if os.Getenv("MCP_MODE") == "true" {
        // Auto-accept all prompts or require --yes flag
        viper.Set("auto_accept", true)
    }
}
```

#### C. Error on Interactive Requirements

Detect TTY absence and return clear errors:

```go
func requireConfirmation() error {
    if !term.IsTerminal(int(os.Stdin.Fd())) {
        return errors.New("This command requires confirmation. Use --confirm=true in MCP mode")
    }
    // Proceed with interactive prompt
}
```

### Challenge 4: Complex Flag Types

**Problem:** Rich Cobra flag types don't have direct JSON Schema equivalents.

**Type Mapping Examples:**

```go
func flagToJSONSchema(flag *pflag.Flag) Property {
    switch flag.Value.Type() {
    case "stringSlice":
        return Property{
            Type:  "array",
            Items: &Property{Type: "string"},
            Description: flag.Usage + " (comma-separated)",
        }
    
    case "stringToString":
        return Property{
            Type: "object",
            AdditionalProperties: &Property{Type: "string"},
            Description: flag.Usage + " (key=value pairs)",
        }
    
    case "duration":
        return Property{
            Type:    "string",
            Pattern: `^\d+(ns|us|ms|s|m|h)$`,
            Description: flag.Usage + " (e.g., '5m', '30s')",
            Examples: []string{"30s", "5m", "1h"},
        }
    
    case "count":
        return Property{
            Type: "integer",
            Minimum: 0,
            Description: flag.Usage + " (incremental count)",
        }
    
    case "ipNet", "ip":
        return Property{
            Type:   "string",
            Format: "ipv4",
            Description: flag.Usage,
        }
    
    default:
        return Property{Type: "string", Description: flag.Usage}
    }
}
```

**Argument Reconstruction:**

```go
func jsonToFlagString(name string, value interface{}, flagType string) string {
    switch flagType {
    case "stringSlice":
        arr := value.([]interface{})
        strs := make([]string, len(arr))
        for i, v := range arr {
            strs[i] = fmt.Sprint(v)
        }
        return fmt.Sprintf("--%s=%s", name, strings.Join(strs, ","))
    
    case "stringToString":
        obj := value.(map[string]interface{})
        pairs := []string{}
        for k, v := range obj {
            pairs = append(pairs, fmt.Sprintf("%s=%v", k, v))
        }
        return fmt.Sprintf("--%s=%s", name, strings.Join(pairs, ","))
    
    case "bool":
        if value.(bool) {
            return fmt.Sprintf("--%s", name)
        }
        return fmt.Sprintf("--%s=false", name)
    
    default:
        return fmt.Sprintf("--%s=%v", name, value)
    }
}
```

### Challenge 5: Output Structure

**Problem:** CLI tools output unstructured text; MCP prefers structured data.

**Solutions:**

#### A. Force JSON Output Mode

```go
func setupMCPEnvironment(cmd *cobra.Command) {
    if os.Getenv("MCP_MODE") == "true" {
        // Try common JSON output flags
        if cmd.Flags().Lookup("output") != nil {
            cmd.Flags().Set("output", "json")
        }
        if cmd.Flags().Lookup("format") != nil {
            cmd.Flags().Set("format", "json")
        }
        if cmd.Flags().Lookup("json") != nil {
            cmd.Flags().Set("json", "true")
        }
    }
}
```

#### B. Parse Common Formats

```go
func structureOutput(raw string, format OutputFormat) interface{} {
    switch format {
    case FormatJSON:
        var obj interface{}
        if err := json.Unmarshal([]byte(raw), &obj); err == nil {
            return obj
        }
    
    case FormatYAML:
        var obj interface{}
        if err := yaml.Unmarshal([]byte(raw), &obj); err == nil {
            return obj
        }
    
    case FormatTable:
        return parseTableOutput(raw)
    
    default:
        return raw // Plain text fallback
    }
    return raw
}
```

#### C. Declare Output Schemas (Optional)

Extend commands with expected output structure:

```go
cmd := &cobra.Command{
    Use:  "list",
    // NEW FIELD:
    OutputSchema: &JSONSchema{
        Type: "array",
        Items: &JSONSchema{
            Type: "object",
            Properties: map[string]Property{
                "id":     {Type: "string"},
                "name":   {Type: "string"},
                "status": {Type: "string"},
            },
        },
    },
}
```

### Challenge 6: Persistent Flags

**Problem:** Persistent flags affect all subcommands—how to represent in flat tools?

**Solution:** Inherit persistent flags in each tool's schema.

```go
func extractFlags(cmd *cobra.Command, schema *JSONSchema) {
    // Local flags
    cmd.Flags().VisitAll(func(flag *pflag.Flag) {
        schema.Properties[flag.Name] = flagToProperty(flag)
        if !flag.DefValue != "" {
            schema.Properties[flag.Name].Default = flag.DefValue
        }
    })
    
    // Persistent flags from ancestors
    for parent := cmd.Parent(); parent != nil; parent = parent.Parent() {
        parent.PersistentFlags().VisitAll(func(flag *pflag.Flag) {
            if _, exists := schema.Properties[flag.Name]; !exists {
                schema.Properties[flag.Name] = flagToProperty(flag)
            }
        })
    }
}
```

**Result:** Each tool includes all applicable flags (local + inherited).

## Usage Patterns

### Basic Integration

```go
package main

import (
    "os"
    "github.com/spf13/cobra"
    "github.com/yourorg/cobra-mcp"
)

func main() {
    rootCmd := &cobra.Command{
        Use:   "myapp",
        Short: "My CLI application",
    }
    
    // Add your commands
    rootCmd.AddCommand(searchCmd, listCmd, deleteCmd)
    
    // Check for MCP mode
    if os.Getenv("MCP_MODE") == "true" {
        if err := cobramcp.ServeMCP(rootCmd, nil); err != nil {
            os.Exit(1)
        }
    } else {
        // Normal CLI execution
        if err := rootCmd.Execute(); err != nil {
            os.Exit(1)
        }
    }
}
```

### With Options

```go
opts := &cobramcp.Options{
    ServerInfo: cobramcp.ServerInfo{
        Name:    "myapp",
        Version: "1.0.0",
    },
    
    // Only expose specific commands
    CommandFilter: func(cmd *cobra.Command) bool {
        return !cmd.Hidden && cmd.Runnable()
    },
    
    // Force JSON output
    PreExecute: func(cmd *cobra.Command) error {
        return cmd.Flags().Set("output", "json")
    },
    
    // Parse structured output
    OutputParser: func(raw string, cmd *cobra.Command) (interface{}, error) {
        return parseJSON(raw)
    },
}

cobramcp.ServeMCP(rootCmd, opts)
```

### With Enhanced Metadata

```go
cmd := cobramcp.NewCommand(&cobra.Command{
    Use:   "search [query]",
    Short: "Search for items",
    Run:   searchFunc,
})

// Add argument metadata
cmd.SetArgs([]cobramcp.ArgSpec{
    {
        Name:        "query",
        Description: "Search query string",
        Type:        "string",
        Required:    true,
    },
})

// Add output schema
cmd.SetOutputSchema(&cobramcp.JSONSchema{
    Type: "array",
    Items: &cobramcp.JSONSchema{
        Type: "object",
        Properties: map[string]cobramcp.Property{
            "title": {Type: "string"},
            "score": {Type: "number"},
        },
    },
})
```

## API Reference

### Core Types

```go
type MCPTool struct {
    Name         string      `json:"name"`
    Description  string      `json:"description,omitempty"`
    InputSchema  JSONSchema  `json:"inputSchema"`
    OutputSchema *JSONSchema `json:"outputSchema,omitempty"`
}

type JSONSchema struct {
    Type                 string              `json:"type"`
    Properties           map[string]Property `json:"properties,omitempty"`
    Required             []string            `json:"required,omitempty"`
    Items                *Property           `json:"items,omitempty"`
    AdditionalProperties *Property           `json:"additionalProperties,omitempty"`
}

type Property struct {
    Type        string      `json:"type"`
    Description string      `json:"description,omitempty"`
    Default     interface{} `json:"default,omitempty"`
    Pattern     string      `json:"pattern,omitempty"`
    Format      string      `json:"format,omitempty"`
    Minimum     *int        `json:"minimum,omitempty"`
    Maximum     *int        `json:"maximum,omitempty"`
    Items       *Property   `json:"items,omitempty"`
    Examples    []string    `json:"examples,omitempty"`
}

type ArgSpec struct {
    Name        string `json:"name"`
    Description string `json:"description"`
    Type        string `json:"type"`
    Required    bool   `json:"required"`
    Variadic    bool   `json:"variadic"`
}
```

### Main Functions

```go
// ServeMCP starts an MCP server for the given Cobra command tree
func ServeMCP(rootCmd *cobra.Command, opts *Options) error

// CommandToMCPTool converts a single command to an MCP tool definition
func CommandToMCPTool(cmd *cobra.Command) MCPTool

// ExecuteMCPTool executes a tool by name with given arguments
func ExecuteMCPTool(rootCmd *cobra.Command, toolName string, args map[string]interface{}) (*MCPResult, error)
```

### Options

```go
type Options struct {
    // Server metadata
    ServerInfo ServerInfo
    
    // Filter which commands to expose
    CommandFilter func(*cobra.Command) bool
    
    // Modify command before execution
    PreExecute func(*cobra.Command) error
    
    // Parse/structure command output
    OutputParser func(string, *cobra.Command) (interface{}, error)
    
    // Handle execution errors
    ErrorHandler func(error) *MCPError
    
    // Logging
    Logger Logger
}
```

## Implementation Checklist

- [ ] Core introspection engine
  - [ ] Command tree walking
  - [ ] Flag extraction and type mapping
  - [ ] Argument parsing from `Use` string
- [ ] JSON Schema generation
  - [ ] Basic types (string, int, bool)
  - [ ] Complex types (slices, maps, durations)
  - [ ] Persistent flag inheritance
- [ ] Execution bridge
  - [ ] Argument reconstruction
  - [ ] Command invocation
  - [ ] Output capture
- [ ] MCP server implementation
  - [ ] JSON-RPC over stdio
  - [ ] `tools/list` handler
  - [ ] `tools/call` handler
  - [ ] Error handling
- [ ] Output parsing
  - [ ] JSON format detection
  - [ ] YAML format detection
  - [ ] Table parsing
- [ ] Enhanced metadata (optional)
  - [ ] ArgSpec annotations
  - [ ] OutputSchema declarations
  - [ ] Custom type converters
- [ ] Testing
  - [ ] Unit tests for type mapping
  - [ ] Integration tests with sample CLIs
  - [ ] MCP protocol compliance tests

## Examples

### Example 1: Simple Search Command

**Cobra Definition:**
```go
searchCmd := &cobra.Command{
    Use:   "search [query]",
    Short: "Search for items",
    Args:  cobra.ExactArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        format, _ := cmd.Flags().GetString("format")
        limit, _ := cmd.Flags().GetInt("limit")
        results := performSearch(args[0], format, limit)
        fmt.Println(results)
    },
}
searchCmd.Flags().StringP("format", "f", "json", "Output format")
searchCmd.Flags().IntP("limit", "l", 10, "Maximum results")
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
        "description": "Query argument"
      },
      "format": {
        "type": "string",
        "description": "Output format",
        "default": "json"
      },
      "limit": {
        "type": "integer",
        "description": "Maximum results",
        "default": 10
      }
    },
    "required": ["query"]
  }
}
```

**MCP Tool Call:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search",
    "arguments": {
      "query": "golang cli",
      "format": "json",
      "limit": 5
    }
  }
}
```

### Example 2: Nested Command

**Cobra Definition:**
```go
dockerCmd := &cobra.Command{Use: "docker"}
containerCmd := &cobra.Command{Use: "container"}
listCmd := &cobra.Command{
    Use:   "list",
    Short: "List containers",
    Run:   listContainers,
}
listCmd.Flags().BoolP("all", "a", false, "Show all containers")

containerCmd.AddCommand(listCmd)
dockerCmd.AddCommand(containerCmd)
```

**Generated MCP Tool:**
```json
{
  "name": "docker_container_list",
  "description": "docker container list: List containers",
  "inputSchema": {
    "type": "object",
    "properties": {
      "all": {
        "type": "boolean",
        "description": "Show all containers",
        "default": false
      }
    }
  }
}
```

### Example 3: Complex Types

**Cobra Definition:**
```go
deployCmd := &cobra.Command{
    Use:   "deploy [app]",
    Short: "Deploy application",
    Args:  cobra.ExactArgs(1),
}
deployCmd.Flags().StringSlice("tags", []string{}, "Image tags")
deployCmd.Flags().StringToString("env", map[string]string{}, "Environment variables")
deployCmd.Flags().Duration("timeout", 5*time.Minute, "Deployment timeout")
```

**Generated MCP Tool:**
```json
{
  "name": "deploy",
  "description": "Deploy application",
  "inputSchema": {
    "type": "object",
    "properties": {
      "app": {
        "type": "string",
        "description": "App argument"
      },
      "tags": {
        "type": "array",
        "items": {"type": "string"},
        "description": "Image tags (comma-separated)"
      },
      "env": {
        "type": "object",
        "additionalProperties": {"type": "string"},
        "description": "Environment variables (key=value pairs)"
      },
      "timeout": {
        "type": "string",
        "pattern": "^\\d+(ns|us|ms|s|m|h)$",
        "description": "Deployment timeout (e.g., '5m', '30s')",
        "default": "5m"
      }
    },
    "required": ["app"]
  }
}
```

## Open Questions

1. **Subcommand strategy**: Should we expose parent commands that have both a `Run` function AND subcommands?
2. **Name collision handling**: How to handle tools with identical flattened names (e.g., `app1_cmd` vs `app1-cmd`)?
3. **Stdin handling**: How should commands that read from stdin behave in MCP mode?
4. **Version compatibility**: Should we version the bridge independently from Cobra versions?
5. **Context propagation**: How to pass MCP request context (timeouts, cancellation) to Cobra commands?

## Future Enhancements

- **Streaming support**: Handle long-running commands with progress updates
- **Resource exposure**: Expose Cobra command help text as MCP resources
- **Prompt generation**: Auto-generate MCP prompts from command examples
- **Caching**: Cache tool definitions for faster startup
- **Custom validators**: Support for custom argument validators beyond standard Cobra types
- **Middleware**: Plugin system for custom transformations

## References

- [Cobra Documentation](https://github.com/spf13/cobra)
- [Model Context Protocol Specification](https://modelcontextprotocol.io)
- [JSON Schema Specification](https://json-schema.org/)
- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)

## Contributing

This is a draft specification. Feedback welcome on:

- Missing use cases or edge cases
- API design improvements
- Implementation approaches
- Alternative solutions to key challenges

## License

TBD
