# Tool Use Guidelines

## Location
All custom tools are located at `~/.tools/tools/<toolName>/tool.ts`.

Each tool lives in its own folder, which can contain:
- `tool.ts` - The main tool script (required)
- `.env` - Credentials/config (optional)
- Helper scripts (optional)
- Any other resources the tool needs

## Execution
Run tools using Bun:
```bash
bun ~/.tools/tools/<toolName>/tool.ts [arguments]
```

**Note:** Bun auto-installs dependencies on first run. No `package.json` or `node_modules` needed - just import what you need and Bun handles the rest.

## Tool Standards

### Folder Structure
```
~/.tools/tools/
├── getIp/
│   └── tool.ts
├── emailTool/
│   ├── tool.ts
│   ├── .env          # Credentials
│   └── sendReport.ts # Helper script (optional)
└── ...
```

### Required JSDoc Header
Every `tool.ts` MUST have a JSDoc comment block at the top:
```typescript
/**
 * @tool toolName
 * @description Brief description of what the tool does
 * @usage bun ~/.tools/tools/toolName/tool.ts <required> [optional]
 * @example bun ~/.tools/tools/toolName/tool.ts arg1 --flag value
 */
```

### Interface
- Accept arguments via `process.argv`
- Output results to stdout
- Use exit code 0 for success, non-zero for failure
- Print errors to stderr

### Credentials (.env)
For tools that need credentials:
1. Create a `.env` file in the tool's folder
2. Load it in the tool using:
```typescript
import { readFileSync } from "fs";
import { join, dirname } from "path";

const toolDir = dirname(import.meta.url.replace("file://", ""));
const envContent = readFileSync(join(toolDir, ".env"), "utf-8");
```

## Helper Script Pattern

For complex operations (like sending formatted emails), AI agents can create temporary helper scripts in the tool folder:

### Example: Sending a formatted email
Instead of struggling with bash escaping, create a helper script:

```typescript
// ~/.tools/tools/emailTool/sendMail.ts
import { execSync } from "child_process";
import { writeFileSync, unlinkSync } from "fs";
import { join, dirname } from "path";

const toolDir = dirname(import.meta.url.replace("file://", ""));

// Write email content to a temp markdown file
const emailContent = `
# Weekly Report

## Summary
- Task 1 completed
- Task 2 in progress

## Details
Here is the **detailed report** with [links](https://example.com).
`;

const mdPath = join(toolDir, "_temp_email.md");
writeFileSync(mdPath, emailContent);

// Send email using the tool
execSync(`bun "${join(toolDir, "tool.ts")}" send --to boss@work.com --subject "Weekly Report" --body "${mdPath}"`, {
  stdio: "inherit"
});

// Cleanup
unlinkSync(mdPath);
```

Then run: `bun ~/.tools/tools/emailTool/sendMail.ts`

### Benefits of Helper Scripts
- No bash escaping issues with quotes, newlines, special characters
- Full TypeScript for complex logic
- Can prepare data, format content, chain operations
- Reusable for recurring tasks

### Naming Convention for Helper Scripts
- Use descriptive camelCase names: `sendReport.ts`, `backupEmails.ts`
- Prefix with underscore for temp files: `_temp_email.md`

## Maintenance Rules

### If a tool fails:
1. Read the tool's source code
2. Identify the issue
3. Fix it using the Edit tool
4. Test the fix
5. **IMPORTANT:** Update `~/.tools/TOOLS.md` if the fix changes usage or behavior

### If a tool is poorly designed:
1. Analyze what's wrong (inefficient, bad error handling, unclear output, etc.)
2. Improve the implementation
3. Update the JSDoc header if usage changes
4. **IMPORTANT:** Update `~/.tools/TOOLS.md` with any changes

### When adding new features to existing tools:
1. Implement the feature
2. Update the JSDoc header in the tool
3. **IMPORTANT:** Update `~/.tools/TOOLS.md` to document the new feature

### If you need a new tool frequently:
1. Create a folder in `~/.tools/tools/` with the tool name
2. Add `tool.ts` with proper JSDoc header
3. Add `.env` if credentials are needed
4. Test that it works
5. **IMPORTANT:** Add the tool to `~/.tools/TOOLS.md` with description and usage

## Reference
See `~/.tools/TOOLS.md` for the list of all available tools with descriptions and usage examples.
