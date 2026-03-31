---
name: test-gap-finder
description: Find source files missing test coverage by comparing source and test directories
user-invocable: true
allowed-tools: [Bash, Read, Grep, Glob]
when-to-use: "When user asks about test coverage, missing tests, or untested code"
---

# Test Gap Finder

Identify source files that lack corresponding test files and highlight untested code paths.

## Steps

1. **Discover project structure:**
   - Find the test directory convention (e.g., `__tests__/`, `*.test.*`, `*.spec.*`, `test/`, `tests/`)
   - Find the source directory (e.g., `src/`, `lib/`, `app/`)
   - Identify the language/framework to understand testing conventions

2. **Map source files to test files:**
   - For each source file, check if a corresponding test file exists
   - Handle naming conventions: `foo.ts` -> `foo.test.ts`, `foo.spec.ts`, `__tests__/foo.ts`
   - Exclude files that typically don't need tests: types, interfaces, constants, index re-exports, config files

3. **Analyze existing tests for shallow coverage:**
   - Check test files for only testing the "happy path"
   - Look for missing edge case coverage (null inputs, empty arrays, error conditions)
   - Identify mocked dependencies that should have integration tests

4. **Generate report:**

### Files Missing Tests Entirely
| Source File | Priority | Reason |
|------------|----------|--------|
| path/to/file | HIGH | Contains business logic with no test |

### Files With Shallow Test Coverage
| Test File | Missing Coverage |
|-----------|-----------------|
| path/to/test | No error case tests, no edge cases |

### Recommendations
- Priority order for writing new tests
- Suggested test scenarios for the highest-priority gaps
- Quick wins (simple files that need simple tests)

Focus on actionable output — prioritize files with business logic, API handlers, and data transformations over simple utility wrappers.
