---
name: writing-tools
description: Use when creating new CLI tools, editing existing tools, or fixing broken tools in ~/.tools/tools/ - provides folder structure, coding standards, JSDoc format, credential handling, helper script patterns, and TOOLS.md maintenance rules
---

# Writing Tools

## Overview

All custom CLI tools live at `~/.tools/tools/<toolName>/tool.ts`. This skill covers how to create, maintain, and document them.

## When to Use

- Creating a new CLI tool
- Fixing or improving an existing tool
- Adding features to a tool
- Tool fails and needs debugging

## Folder Structure

```
~/.tools/tools/
  toolName/
    tool.ts           # Main script (required)
    .env              # Credentials (optional)
    helperScript.ts   # Helper scripts (optional)
```

## Required JSDoc Header

Every `tool.ts` MUST start with:

```typescript
/**
 * @tool toolName
 * @description Brief description of what the tool does
 * @usage bun ~/.tools/tools/toolName/tool.ts <required> [optional]
 * @example bun ~/.tools/tools/toolName/tool.ts arg1 --flag value
 */
```

## Tool Interface Standards

- Accept arguments via `process.argv`
- Output results to stdout
- Errors to stderr
- Exit code 0 = success, non-zero = failure
- Support `--help` flag
- Support `--json` flag where applicable
- Bun auto-installs dependencies -- no package.json needed, just import

## Credential Handling

```typescript
import { readFileSync } from "fs";
import { join, dirname } from "path";

const toolDir = dirname(import.meta.url.replace("file://", ""));
const envContent = readFileSync(join(toolDir, ".env"), "utf-8");
```

## Helper Script Pattern

For complex operations (formatted emails, multi-step workflows), create helper scripts instead of fighting bash escaping:

```typescript
// ~/.tools/tools/emailTool/sendReport.ts
import { execSync } from "child_process";
import { writeFileSync, unlinkSync } from "fs";
import { join, dirname } from "path";

const toolDir = dirname(import.meta.url.replace("file://", ""));

const content = `# Report\n- Item 1\n- Item 2`;
const mdPath = join(toolDir, "_temp.md");
writeFileSync(mdPath, content);

execSync(`bun "${join(toolDir, "tool.ts")}" send --to user@example.com --subject "Report" --body "${mdPath}"`, {
  stdio: "inherit"
});

unlinkSync(mdPath);
```

Naming: camelCase for helpers (`sendReport.ts`), underscore prefix for temp files (`_temp.md`).

## Documentation Rules

After ANY tool change, you MUST update docs:

### New tool:
1. Create `~/.tools/tools/toolName/tool.ts` with JSDoc header
2. Add `.env` if credentials needed
3. Test it works
4. Add entry to `~/.tools/TOOLS.md` (full docs) AND `~/.tools/TOOLS_QUICK.md` (one-line reference)

### Modified tool:
1. Update JSDoc header if usage changed
2. Update `~/.tools/TOOLS.md` if behavior/options changed
3. Update `~/.tools/TOOLS_QUICK.md` if description/usage pattern changed

### Broken tool:
1. Read tool source, identify issue, fix it
2. Test the fix
3. Update docs if the fix changes usage

## Common Mistakes

- Forgetting `--help` support
- Not updating TOOLS.md after changes
- Using `process.cwd()` instead of `import.meta.url` for tool-relative paths
- Hardcoding paths instead of using `toolDir`
- Not printing errors to stderr
