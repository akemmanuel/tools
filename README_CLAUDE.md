# opencode-tools for Claude Code

> Setup instructions for using opencode-tools with Claude Code (Anthropic's official CLI)

## Overview

Claude Code uses a different configuration system than opencode:

| Feature | opencode | Claude Code |
|---------|----------|-------------|
| Main config | `~/.config/opencode/opencode.json` | `~/.claude/settings.json` |
| Custom instructions | `"instructions": ["file.md"]` | `CLAUDE.md` files + `@imports` |
| Permissions | `"permission": { "external_directory": {...} }` | `"permissions": { "allow": [...] }` |

## Setup

### 1. Copy tool files to ~/.tools

```bash
mkdir -p ~/.tools/tools
cp TOOLS.md TOOL_USE.md ~/.tools/
```

### 2. Create `~/.claude/settings.json`

```bash
mkdir -p ~/.claude
```

Create `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Read(~/.tools/**)",
      "Edit(~/.tools/**)",
      "Bash(bun ~/.tools/tools/*)"
    ],
    "additionalDirectories": ["~/.tools/"]
  }
}
```

This grants Claude Code:
- **Read access** to all files in `~/.tools/`
- **Edit access** to modify/fix tools on the fly
- **Bash execution** for running tools with `bun`

### 3. Create `~/.claude/CLAUDE.md`

Create `~/.claude/CLAUDE.md`:

```markdown
# Global Instructions

## Custom Tools
See @~/.tools/TOOLS.md for available custom tools.
See @~/.tools/TOOL_USE.md for tool usage guidelines.

## Development Rules
- Never run dev servers (bun run dev, bun run start) - assume user is running them
```

The `@` syntax tells Claude Code to automatically import content from referenced files.

## How It Works

1. Claude Code reads `~/.claude/CLAUDE.md` at startup
2. The `@~/.tools/TOOLS.md` reference pulls in your tool catalog
3. The `@~/.tools/TOOL_USE.md` reference provides usage guidelines
4. Permissions in `settings.json` allow Claude to execute and maintain tools

## Configuration Hierarchy

Claude Code supports multiple configuration scopes (highest to lowest priority):

| Scope | Location | Purpose |
|-------|----------|---------|
| Managed | `/etc/claude-code/managed-settings.json` | Enterprise policies |
| Command-line | CLI flags | Session overrides |
| Local | `.claude/settings.local.json` | Personal project settings (gitignored) |
| Project | `.claude/settings.json` | Team-shared project settings |
| User | `~/.claude/settings.json` | Global defaults |

## Permission Rule Syntax

| Pattern | Meaning |
|---------|---------|
| `Bash` | All bash commands |
| `Bash(bun ~/.tools/tools/*)` | Tool execution only |
| `Read(~/.tools/**)` | Read anything in ~/.tools/ recursively |
| `Edit(~/.tools/**)` | Edit anything in ~/.tools/ recursively |

## CLAUDE.md File Locations

| Location | Scope | Shared with Team? |
|----------|-------|-------------------|
| `~/.claude/CLAUDE.md` | User (global) | No |
| `CLAUDE.md` (project root) | Project | Yes |
| `.claude/CLAUDE.md` | Project | Yes |
| `CLAUDE.local.md` | Local (personal) | No (gitignored) |

## Useful Commands

| Command | Purpose |
|---------|---------|
| `/init` | Generate starter CLAUDE.md from project analysis |
| `/config` | Open configuration interface |
| `/permissions` | Manage tool permissions |
| `/help` | List all available commands |

## Differences from opencode

1. **No `instructions` array** - Use `CLAUDE.md` files with `@imports` instead
2. **Permission syntax** - Uses `allow`/`deny` arrays with tool-specific patterns
3. **Multiple config scopes** - User, project, local, and managed settings
4. **Built-in `@import`** - Reference external files directly in CLAUDE.md
