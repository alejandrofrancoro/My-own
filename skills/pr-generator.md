---
name: pr-generator
description: Generate a well-structured PR description from current git changes
user-invocable: true
allowed-tools: [Bash, Read, Grep, Glob]
when-to-use: "When user wants to create a pull request or generate a PR description"
---

# PR Description Generator

Generate a comprehensive pull request description from the current branch changes.

## Steps

1. **Gather context:**
   - Run `git log main..HEAD --oneline` (or appropriate base branch) to get all commits
   - Run `git diff main..HEAD --stat` to see changed files summary
   - Run `git diff main..HEAD` to see the actual changes
   - Read any modified files that need deeper understanding

2. **Analyze the changes:**
   - Identify the type: feature, bugfix, refactor, docs, tests, chore
   - Understand the "why" behind the changes (from commit messages and code context)
   - Note any breaking changes or migration requirements
   - Identify files that reviewers should focus on

3. **Generate the PR description:**

```markdown
## Summary
[1-3 bullet points explaining what this PR does and why]

## Changes
[Categorized list of changes by area/component]

## Key Files to Review
[Most important files for reviewers to focus on, with brief context]

## Testing
[How to test these changes - steps, commands, or checklist]

## Breaking Changes
[Any breaking changes, migration steps, or deployment notes — or "None"]
```

4. **Output the description** ready to copy-paste into a PR form.

If the user wants to actually create the PR, suggest using `gh pr create` with the generated description.
