# opencode-tools for Claude Code

> Setup instructions for using opencode-tools with Claude Code (Anthropic's official CLI)

## Overview

Claude Code uses a different configuration system than opencode:

| Feature | opencode | Claude Code |
|---------|----------|-------------|
| Main config | `~/.config/opencode/config.json` | `~/.claude/settings.json` |
| Custom instructions | `"instructions": ["file.md"]` | `AGENTS.md` files + `@imports` |
| Permissions | `"permission": { "external_directory": {...} }` | `"permissions": { "allow": [...] }` |
| On-demand skills | `~/.opencode/skill/` | `~/.claude/skills/` (via AGENTS.md) |

## Setup

### 1. Copy tool files to ~/.tools

```bash
mkdir -p ~/.tools/tools
cp TOOLS_QUICK.md ~/.tools/
cp TOOLS.md ~/.tools/
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

### 3. Create `~/.claude/AGENTS.md`

```markdown
# Global Instructions

## Custom Tools
See @~/.tools/TOOLS_QUICK.md for available custom tools.
Run `--help` on any tool for detailed options.
Full docs: @~/.tools/TOOLS.md
```

Note: The `@` syntax tells Claude Code to automatically import content from referenced files.

### 4. (Optional) Add the writing-tools skill

```bash
mkdir -p ~/.claude/skills/writing-tools
cp skill/writing-tools/SKILL.md ~/.claude/skills/writing-tools/
```

## How It Works

1. Claude Code reads `~/.claude/AGENTS.md` at startup
2. The `@~/.tools/TOOLS_QUICK.md` reference pulls in the compact tool catalog (~2KB)
3. `TOOLS.md` is available on-demand for full documentation
4. Permissions in `settings.json` allow Claude to execute and maintain tools
5. When creating/editing tools, the agent references the writing-tools skill

## Token Optimization

The compact `TOOLS_QUICK.md` (~2KB) replaces the previous approach of loading full docs (~50KB) every session. This is a 96% reduction in per-message token overhead.

| What | When | Size |
|------|------|------|
| `TOOLS_QUICK.md` (compact reference table) | Every session | ~2KB (~560 tokens) |
| `writing-tools` skill | Only when creating/editing tools | ~3KB (~813 tokens) |
| `TOOLS.md` (full documentation) | On-demand | ~50KB |
| `tool.ts --help` | Only when using a specific tool | Varies |

For detailed tool options: `tool.ts --help`
For creating/editing tools: load the `writing-tools` skill

## Configuration Hierarchy

Claude Code supports multiple configuration scopes (highest to lowest priority):

| Scope | Location | Purpose |
|-------|----------|---------|
| Managed | `/etc/claude-code/managed-settings.json` | Enterprise policies |
| Command-line | CLI flags | Session overrides |
| Local | `.claude/settings.local.json` | Personal project settings (gitignored) |
| Project | `.claude/settings.json` | Team-shared project settings |
| User | `~/.claude/settings.json` | Global defaults |

## AGENTS.md File Locations

| Location | Scope | Shared with Team? |
|----------|-------|-------------------|
| `~/.claude/AGENTS.md` | User (global) | No |
| `AGENTS.md` (project root) | Project | Yes |
| `.claude/AGENTS.md` | Project | Yes |
| `AGENTS.local.md` | Local (personal) | No (gitignored) |
