# opencode-tools

> **Experimental Concept** - Extending AI coding agents with custom CLI tools

This is an experimental concept for giving AI coding agents access to custom Bun-powered CLI tools through instruction files.

**Not affiliated with [opencode](https://opencode.ai)** - This is an independent implementation of this concept that happens to work well with opencode.

## Why Not Just Use MCP?

This approach has several advantages over traditional MCP servers:

### Faster Execution
With MCP, the AI often needs multiple round-trips. For example, to delete spam emails, an MCP-based agent would: list emails → load IDs into context → delete one by one. With this approach, the AI can write a helper script that finds and deletes all emails from `no-reply@example.com` in a single execution - no round-trips, no loading IDs into context.

### Cheaper (Less Context)
- Tool descriptions are short and concise - the AI only reads detailed docs when needed
- No bloated tool schemas loaded into every request
- Helper scripts process data locally instead of passing everything through context

### Self-Improving
The AI can fix, improve, and extend its own tools on the fly:
- Tool broken? The AI reads the source, fixes it, done
- Missing a feature? The AI adds it
- No need to restart anything or touch config files

### Zero-Config Development
- No MCP server configuration in `opencode.json`
- No restarting the agent to add new tools
- The AI creates tools as needed for repetitive tasks (web scraping, data processing, etc.)
- Tools are just TypeScript files - dead simple to write and maintain

## The Concept

The idea is simple: AI coding agents can read instruction files and execute bash commands. By combining:

- **Instruction files** that teach the agent about available tools
- **Simple CLI tools** (TypeScript scripts run with Bun)
- **Permission rules** that allow access to the tools directory

...you get an AI agent that can use your custom tools.

This pattern works with any AI agent that supports custom instructions and bash execution. This repo provides a ready-to-use implementation for opencode.

## Setup (opencode)

### 1. Copy files to ~/.tools

```bash
mkdir -p ~/.tools/tools
cp TOOLS.md TOOL_USE.md ~/.tools/
```

### 2. Configure opencode

Add to `~/.config/opencode/opencode.json`:

```json
{
  "instructions": ["~/.tools/TOOLS.md", "~/.tools/TOOL_USE.md"],
  "permission": {
    "external_directory": {
      "~/.tools/**": "allow"
    }
  }
}
```

That's it. The agent will now:
- Read `TOOLS.md` to learn what tools are available
- Read `TOOL_USE.md` to understand how to use and maintain them
- Have permission to execute tools in `~/.tools/`

## Creating Your Own Tools

Tools live in `~/.tools/tools/<toolName>/tool.ts`. Each tool is a simple TypeScript script that Bun runs directly (dependencies are auto-installed).

Example structure:
```
~/.tools/
├── TOOLS.md          # Documents available tools (agent reads this)
├── TOOL_USE.md       # Guidelines for using/creating tools
└── tools/
    └── myTool/
        ├── tool.ts   # The tool script
        └── .env      # Optional credentials
```

See [TOOL_USE.md](./TOOL_USE.md) for full guidelines on creating tools.

## Adapting for Other AI Agents

The same concept can be adapted to other AI coding agents:

1. Point your agent's instruction/rules config to the markdown files
2. Ensure the agent has permission to execute bash in `~/.tools/`
3. The agent will learn about and use your tools automatically

The key files are:
- `TOOLS.md` - Catalog of available tools with usage examples
- `TOOL_USE.md` - Guidelines for how the agent should use and maintain tools

## Known Limitations

This can of course be improved. Currently, the AI loads `TOOLS.md` and `TOOL_USE.md` into context on every session, which takes up some tokens. It's still much better than MCP's constant schema overhead, but there's room for optimization (e.g., lazy loading, summarization, etc.).

## Contributing

If you have ideas for enhancements, optimizations, or new features - please create a PR! I'm open to new ideas and improvements.
