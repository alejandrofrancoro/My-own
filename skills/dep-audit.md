---
name: dep-audit
description: Audit project dependencies for vulnerabilities, outdated packages, and bloat
user-invocable: true
allowed-tools: [Bash, Read, Grep, Glob]
when-to-use: "When user asks about dependencies, outdated packages, vulnerabilities, or package audit"
---

# Dependency Auditor

Perform a comprehensive audit of project dependencies.

## Steps

1. **Detect package manager:** npm, yarn, pnpm, bun, pip, cargo, go mod, etc.

2. **Security vulnerabilities:**
   - Run the appropriate audit command (`npm audit`, `yarn audit`, `pip-audit`, `cargo audit`, etc.)
   - Parse and summarize findings by severity
   - Provide fix commands for each vulnerability

3. **Outdated packages:**
   - Run outdated check (`npm outdated`, `yarn outdated`, etc.)
   - Categorize: patch updates (safe), minor updates (usually safe), major updates (breaking)
   - Flag packages that are 2+ major versions behind

4. **Dependency bloat:**
   - Check for duplicate packages in lockfile
   - Identify large dependencies that might have lighter alternatives
   - Find unused dependencies (grep imports against package.json deps)

5. **License compliance:**
   - List dependency licenses
   - Flag any copyleft licenses (GPL) in non-GPL projects
   - Flag packages with no license

## Output

### Critical Vulnerabilities
[Immediate action items with fix commands]

### Outdated Packages
| Package | Current | Latest | Type | Risk |
|---------|---------|--------|------|------|

### Unused Dependencies
[Packages in package.json but not imported anywhere]

### Recommendations
[Prioritized action items]
