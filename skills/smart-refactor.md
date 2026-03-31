---
name: smart-refactor
description: Analyze and refactor code for better patterns, performance, and maintainability
user-invocable: true
allowed-tools: [Bash, Read, Edit, Write, Grep, Glob]
model: opus
effort: high
arguments:
  - name: target
    description: File or directory to refactor (e.g., src/utils/)
when-to-use: "When user asks to refactor, clean up, or improve code quality"
---

# Smart Refactor

Analyze the target code and perform intelligent refactoring. This is NOT about cosmetic changes — focus on meaningful improvements.

## Target: $ARGUMENTS

## Analysis Phase

1. **Read all files** in the target path
2. **Identify issues** (prioritized):
   - **Bugs**: Logic errors, race conditions, unhandled edge cases
   - **Performance**: N+1 queries, unnecessary re-renders, O(n^2) where O(n) is possible
   - **Duplication**: Copy-pasted logic that should be extracted
   - **Complexity**: Functions doing too many things, deep nesting
   - **Naming**: Misleading names that don't match behavior
   - **Dead code**: Unreachable branches, unused exports

3. **Check for existing patterns**: Before creating new abstractions, look for existing utilities, helpers, or patterns in the codebase that should be reused.

## Refactoring Phase

4. **Make changes** following these rules:
   - One concern at a time — don't mix bug fixes with style changes
   - Preserve all existing behavior (unless fixing a bug)
   - Don't add abstractions for single-use code
   - Don't add comments for self-evident code
   - Keep the diff minimal — change what matters, leave the rest

5. **Verify** after each change:
   - Run any available tests
   - Check for type errors if TypeScript
   - Ensure imports are correct

## Output
Summarize what was changed and why, with before/after for the most impactful changes.
