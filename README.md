# opencode-tools

> **Experimental Concept** - Extending AI coding agents with custom CLI tools

This is an experimental concept for giving AI coding agents access to custom Bun-powered CLI tools through instruction files.

**Not affiliated with [opencode](https://opencode.ai)** - This is an independent implementation that happens to work well with opencode.

## Why Not Just Use MCP?

### Faster Execution
With MCP, the AI often needs multiple round-trips. With this approach, the AI can write a helper script that processes everything in a single execution - no round-trips, no loading IDs into context.

### Cheaper (Less Context)
- A compact reference table (`TOOLS_QUICK.md`, ~2KB) is loaded every session instead of full documentation (~50KB)
- Full docs (`TOOLS.md`) and tool-creation guidelines are loaded on-demand via skills
- Helper scripts process data locally instead of passing everything through context

### Self-Improving
The AI can fix, improve, and extend its own tools on the fly. Tool broken? The AI reads the source, fixes it, done. No restart needed.

### Zero-Config Development
No MCP server configuration, no restarting the agent. The AI creates tools as needed - they're just TypeScript files.

## The Concept

AI coding agents can read instruction files and execute bash commands. By combining:

- **A compact tool reference** (`TOOLS_QUICK.md`) loaded every session (~2KB)
- **Full tool documentation** (`TOOLS.md`) available on-demand
- **A skill for writing tools** loaded on-demand only when creating/editing tools
- **Simple CLI tools** (TypeScript scripts run with Bun)
- **Permission rules** that allow access to the tools directory

...you get an AI agent that can use your custom tools with minimal token overhead.

## Setup (opencode)

### 1. Copy files to ~/.tools

```bash
mkdir -p ~/.tools/tools
cp TOOLS_QUICK.md ~/.tools/
cp TOOLS.md ~/.tools/
```

### 2. Create the writing-tools skill

Copy the skill for creating/maintaining tools (loaded on-demand, not every session):

```bash
mkdir -p ~/.opencode/skill/writing-tools
cp skill/writing-tools/SKILL.md ~/.opencode/skill/writing-tools/
```

### 3. Configure opencode

Add to `~/.config/opencode/config.json`:

```json
{
  "instructions": ["~/.tools/TOOLS_QUICK.md"],
  "permission": {
    "external_directory": {
      "~/.tools/**": "allow"
    }
  }
}
```

That's it. The agent will now:
- Read `TOOLS_QUICK.md` every session to know what tools are available (compact, ~2KB)
- Use `TOOLS.md` for full documentation when needed (on-demand)
- Load the `writing-tools` skill on-demand when it needs to create or fix tools
- Run `tool.ts --help` for detailed options on any specific tool
- Have permission to execute tools in `~/.tools/`

## File Structure

```
~/.tools/
├── TOOLS_QUICK.md        # Compact tool reference table (loaded every session, ~2KB)
├── TOOLS.md              # Full tool documentation (on-demand)
└── tools/
    └── myTool/
        ├── tool.ts       # The tool script
        └── .env          # Optional credentials
```

## Creating Your Own Tools

Tools live in `~/.tools/tools/<toolName>/tool.ts`. Each tool is a simple TypeScript script that Bun runs directly (dependencies are auto-installed).

To create or edit tools, the agent loads the `writing-tools` skill which provides:
- Folder structure and coding standards
- Required JSDoc header format
- Credential handling patterns
- Helper script patterns for complex operations
- Documentation maintenance rules (updating both `TOOLS_QUICK.md` and `TOOLS.md`)

## Token Optimization

The key insight: **don't load everything every session**.

| What | When | Size |
|------|------|------|
| `TOOLS_QUICK.md` (compact reference table) | Every session | ~2KB (~560 tokens) |
| `writing-tools` skill | Only when creating/editing tools | ~3KB (~813 tokens) |
| `TOOLS.md` (full documentation) | On-demand | ~50KB |
| `tool.ts --help` | Only when using a specific tool | Varies |

**Before optimization:** ~50KB loaded every message (~12,600 tokens)
**After optimization:** ~2KB loaded every message (~560 tokens) = **96% reduction**

## Adapting for Other AI Agents

The same concept can be adapted to other AI coding agents:

1. Point your agent's instruction/rules config to `TOOLS_QUICK.md`
2. Ensure the agent has permission to execute bash in `~/.tools/`
3. Provide the writing-tools skill (or equivalent guidelines) for tool creation
4. The agent will learn about and use your tools automatically

See [README_CLAUDE.md](./README_CLAUDE.md) for Claude Code-specific setup.

## Contributing

If you have ideas for enhancements, optimizations, or new features - please create a PR!
