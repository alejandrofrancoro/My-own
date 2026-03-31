# Claude Code Source Code: Hidden Gems & Power User Guide

## Context
The Claude Code source (512K+ lines of TypeScript) was made public on 2026-03-31. This document distills the non-obvious features, hidden mechanics, and actionable techniques revealed by the codebase that can dramatically improve how you use Claude Code.

---

## 1. SKILL SYSTEM — Build Your Own Power Tools

**What the code reveals:** Skills are far more powerful than simple prompt templates. They're full agent configurations.

### Skill Frontmatter Capabilities
```markdown
---
name: my-skill
description: What the skill does
user-invocable: true
allowed-tools: [Bash, Read, Edit, Write, Grep, Glob]
model: opus
effort: high
hooks:
  - event: PostToolUse
    command: "npm test"
arguments:
  - name: target
    description: The file to process
paths: ["src/**/*.ts"]           # Only activate for these files
when-to-use: "When user asks to refactor TypeScript"
---
```

### Key Insights for Skills
- **`allowed-tools`**: Restrict which tools a skill can use (security + focus)
- **`model`**: Override the model per-skill (use haiku for simple tasks, opus for complex)
- **`effort`**: Hint computational budget
- **`paths`**: Git-ignore style patterns — skill only activates when matching files are involved
- **`when-to-use`**: Semantic trigger — Claude proactively suggests the skill
- **`arguments`**: Define parameters with `$ARGUMENTS` substitution in the prompt
- **`hooks`**: Attach automation (e.g., run tests after skill completes)
- Skills can run as `inline` (same context) or `fork` (isolated sub-agent)

### Skill Ideas Based on Source Patterns
1. **Auto-reviewer**: Skill that runs on `paths: ["src/**"]` with `when-to-use: "When code changes are ready for review"`, uses Grep/Read to check patterns, runs tests via Bash
2. **Migration generator**: Skill restricted to `allowed-tools: [Read, Write, Bash]` that reads schema files and generates migrations
3. **Security auditor**: Skill with `effort: high` and `model: opus` that greps for common vulnerability patterns

---

## 2. HOOKS — The Automation Layer Most People Ignore

**What the code reveals:** 26 hook events, 4 hook types, conditional filtering.

### All 26 Hook Events
| Event | When It Fires |
|-------|--------------|
| `PreToolUse` | Before any tool executes |
| `PostToolUse` | After tool succeeds |
| `PostToolUseFailure` | After tool fails |
| `UserPromptSubmit` | When user sends a message |
| `SessionStart` / `SessionEnd` | Session lifecycle |
| `Stop` / `StopFailure` | Agent stop events |
| `SubagentStart` / `SubagentStop` | Sub-agent lifecycle |
| `PreCompact` / `PostCompact` | Context compaction |
| `Notification` | System notifications |
| `PermissionRequest` / `PermissionDenied` | Permission events |
| `TaskCreated` / `TaskCompleted` | Task lifecycle |
| `Setup` | Initial setup |
| `TeammateIdle` | Teammate idle detection |
| `Elicitation` / `ElicitationResult` | User question events |
| `ConfigChange` | Settings change |
| `InstructionsLoaded` | CLAUDE.md loaded |
| `CwdChanged` | Working directory change |
| `FileChanged` | File modification |
| `WorktreeCreate` / `WorktreeRemove` | Worktree lifecycle |

### 4 Hook Types
1. **Command hooks**: Run shell commands
2. **Prompt hooks**: LLM evaluation with custom model
3. **HTTP hooks**: POST to webhooks
4. **Agent hooks**: Run a verification agent

### Hook Features Most People Miss
- **`if` condition**: Filter using permission rule syntax (e.g., `Bash(git *)` only fires for git commands)
- **`once`**: Auto-removes after first execution
- **`async`**: Fire in background without blocking
- **`asyncRewake`**: Background execution that blocks on exit code 2 (error)
- **`statusMessage`**: Custom spinner text during execution

### Practical Hook Examples
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "eslint --fix $TOOL_INPUT_FILE",
          "if": "Write(src/**/*.ts)"
        }]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "echo 'BLOCKED' && exit 1",
          "if": "Bash(rm -rf *)"
        }]
      }
    ]
  }
}
```

---

## 3. MEMORY SYSTEM — How It Actually Works

### Memory Constants (from source)
- **Per-file cap**: 4,096 bytes (truncated preserving frontmatter + opening)
- **Per-file line cap**: 200 lines
- **Session memory budget**: 60KB (~3 full injections before cap)
- **MEMORY.md index**: 200 lines max, then truncated

### Auto-Memory Extraction (Background Process)
- Runs as a perfect fork of main conversation
- Skips turns where the main agent already wrote memories
- Limited turn budget: all reads in turn 1, all writes in turn 2
- Tools available: Read, Grep, Glob, read-only Bash, Edit/Write (memory dir only)

### Auto-Dream (Memory Consolidation)
- **Gate**: `tengu_onyx_plover` feature flag
- Triggers when: hours since last consolidation >= threshold AND session count >= minimum
- Runs `/dream` as a forked sub-agent
- Consolidates fragmented memories into coherent summaries
- Lock-based to prevent parallel runs

### Best Practices from Source
- Create individual memory files (not one giant file) — they're managed separately
- Use the 4-type taxonomy: `user`, `feedback`, `project`, `reference`
- Keep MEMORY.md under 200 lines — it's an index, not storage
- Memory files support frontmatter with `name`, `description`, `type`

---

## 4. CONTEXT & PROMPT CACHING — How to Get Faster Responses

### How Prompt Caching Works Internally
The system prompt is split into up to 4 blocks with different cache scopes:
1. **Attribution header** (no cache — changes per request)
2. **CLI prefix** (org cache — "You are Claude Code...")
3. **Static boundary content** (global cache — only for 1st party API)
4. **Dynamic content** (no cache — after boundary marker)

### What This Means for You
- **CLAUDE.md files participate in prompt caching** — stable instructions get cached
- Keep CLAUDE.md content stable between sessions (don't change it every time)
- Put volatile instructions at the END of CLAUDE.md (after stable content)
- Tool schemas are cached per session — they won't change mid-conversation

### Token Efficiency
- Tool search defers loading (reduces initial tool list)
- Eager input streaming prevents hangs on large tool parameters
- Max 3 recovery attempts for `max_output_tokens` errors (auto-continues)

---

## 5. COORDINATOR MODE — Multi-Agent Power

### How to Enable
```bash
export CLAUDE_CODE_COORDINATOR_MODE=1
```

### What It Unlocks
- Spawns autonomous **worker agents** via the `agent` tool
- Workers get: Bash, File I/O, MCP tools, Skills
- Workers are **fully autonomous** — no checkpoint approvals
- Results arrive as `<task-notification>` in the conversation
- Continue workers with `SendMessage` to reuse loaded context
- Stop workers mid-flight with `TaskStop` to redirect

### Scratchpad Directory
- Gate: `tengu_scratch` feature flag
- Shared file-based knowledge between workers
- Workers can read/write without permission prompts

---

## 6. HIDDEN SETTINGS MOST PEOPLE DON'T KNOW ABOUT

### settings.json Power Options
```json
{
  "worktree": {
    "symlinkDirectories": ["node_modules", ".cache"],
    "sparsePaths": ["packages/core", "packages/utils"]
  },
  "claudeMdExcludes": ["**/vendor/**", "**/generated/**"],
  "autoMemoryEnabled": true,
  "autoMemoryDirectory": "~/my-memories",
  "respectGitignore": true,
  "cleanupPeriodDays": 30,
  "defaultView": "transcript",
  "prefersReducedMotion": true,
  "showThinkingSummaries": true,
  "attribution": {
    "commit": "Custom commit message trailer",
    "pr": "Custom PR attribution"
  }
}
```

### Key Environment Variables
| Variable | Purpose |
|----------|---------|
| `BASH_DEFAULT_TIMEOUT_MS` | Default bash timeout |
| `BASH_MAX_TIMEOUT_MS` | Max allowed bash timeout |
| `BASH_MAX_OUTPUT_LENGTH` | Truncation limit for bash output |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max tokens per response |
| `MAX_THINKING_TOKENS` | Thinking budget |
| `MAX_MCP_OUTPUT_TOKENS` | MCP tool output limit |
| `MCP_TIMEOUT` / `MCP_TOOL_TIMEOUT` | MCP timeouts |
| `CLAUDE_CODE_SUBAGENT_MODEL` | Override sub-agent model |
| `ENABLE_TOOL_SEARCH` | Enable deferred tool discovery |

---

## 7. HIDDEN/NOTABLE SLASH COMMANDS

| Command | What It Does |
|---------|-------------|
| `/dream` | Background memory consolidation — merges fragmented memories |
| `/ultraplan` | Remote multi-agent planning (30-min timeout, uses Claude on the web) |
| `/btw` | Side-panel question during execution (doesn't interrupt current work) |
| `/x402` | Cryptocurrency wallet + USDC payments for 402-gated APIs |
| `/thinkback` | "Year in Review" animation of your Claude Code usage |
| `/compact` | Manual context compaction (3 modes: full, partial, prefix) |
| `/context` | Visualize what's in your current context window |
| `/plan` | Structured planning with effort levels |

---

## 8. BASH TOOL INTERNALS — Why Certain Commands Behave Differently

The BashTool classifies every command semantically:
- **Read commands**: cat, head, tail, jq, awk → collapsed in UI
- **Search commands**: find, grep, rg, locate → collapsed in UI
- **List commands**: ls, tree, du → collapsed in UI
- **Neutral**: echo, printf, true, false

It also:
- Parses `sed` commands to detect file modifications (blocks in read-only mode)
- Validates against UNC paths on Windows (prevents NTLM credential leaks)
- Supports sandbox detection and conditional activation

---

## 9. MCP DEDUPLICATION — How Conflicts Are Resolved

When multiple MCP servers overlap:
1. **Manual servers always win** over plugin servers
2. **Plugin servers** are deduped against each other (first loaded wins)
3. **claude.ai connectors** are suppressed when matching manual server is enabled
4. Signatures are content-based: `stdio:${command_array}` or `url:${original_url}`

### Enterprise Controls
- Allowlist/denylist by: server name, exact command, URL pattern (wildcards)
- Denylist takes precedence over allowlist
- Empty allowlist = no servers allowed

---

## 10. THE BUDDY SYSTEM (Easter Egg)

Every user gets a procedurally generated companion:
- **19 species**: duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk
- **5 rarities**: common (60%), uncommon (25%), rare (10%), epic (4%), legendary (1%)
- **1% shiny chance**
- Deterministic from your user ID (same companion every session)
- Has stats: DEBUGGING, PATIENCE, CHAOS, WISDOM, SNARK

---

## 11. COMPACTION — How Long Sessions Stay Useful

Three compaction modes:
1. **Full**: Summarizes entire conversation into 9 sections
2. **Partial**: Only summarizes recent messages (keeps older context)
3. **Prefix**: Summarizes older messages, places before newer ones

The 9 summary sections:
1. Original request intent
2. Key concepts and definitions
3. Files read/modified
4. Errors encountered
5. Problem-solving approaches
6. Important user messages
7. Pending tasks
8. Current work state
9. Next step

**Pro tip**: Uses an `<analysis>` scratchpad that's stripped before caching — improves summary quality without wasting context.

---

## 12. ACTIONABLE SKILL IDEAS TO BUILD

Based on patterns in the source code:

1. **Pre-commit validator**: Hook on `PreToolUse` for `Bash(git commit*)`, runs linting + tests
2. **Smart file watcher**: Hook on `FileChanged` that auto-runs relevant tests
3. **Context preloader**: Skill with `paths: ["src/**"]` that pre-reads architecture docs
4. **PR template generator**: Skill that reads git diff and generates structured PR descriptions
5. **Dependency auditor**: Skill restricted to `[Read, Grep, Bash]` that checks for outdated/vulnerable deps
6. **Migration checker**: Hook on `PostToolUse` for `Write(src/migrations/*)` that validates migration files
7. **Auto-documenter**: Skill with `when-to-use: "When user creates a new module"` that generates initial docs
8. **Test gap finder**: Skill that compares source files to test files and identifies missing coverage

---

## Verification
- Skills: Create a `.claude/skills/` directory with a test skill and invoke it with `/skill-name`
- Hooks: Add a hook to `~/.claude/settings.json` and verify it fires with a simple echo command
- Memory: Check `~/.claude/projects/*/memory/` for auto-generated memories
- Environment variables: Set variables and verify behavior changes
