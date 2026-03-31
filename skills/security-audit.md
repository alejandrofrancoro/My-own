---
name: security-audit
description: Scan codebase for common security vulnerabilities (OWASP Top 10, secrets, injection risks)
user-invocable: true
allowed-tools: [Bash, Read, Grep, Glob]
model: opus
effort: high
when-to-use: "When user asks to audit security, check for vulnerabilities, or review code safety"
---

# Security Audit Skill

Perform a comprehensive security audit of the current project. Check for:

## 1. Secrets & Credentials
- Grep for hardcoded API keys, tokens, passwords, and secrets in source files
- Check for .env files committed to git (check .gitignore)
- Look for private keys, certificates, or credentials in the repo
- Patterns: `password\s*=`, `api_key`, `secret`, `token`, `Bearer `, `-----BEGIN`, base64-encoded strings that look like keys

## 2. Injection Vulnerabilities
- SQL injection: raw string concatenation in queries (no parameterized queries)
- Command injection: unsanitized input passed to exec/spawn/system calls
- XSS: unescaped user input rendered in HTML/templates
- Path traversal: user input in file paths without sanitization

## 3. Authentication & Authorization
- Check for missing auth middleware on routes
- Look for hardcoded JWT secrets
- Check session configuration (httpOnly, secure, sameSite flags)
- Look for privilege escalation risks (missing role checks)

## 4. Dependencies
- Run `npm audit` or equivalent if package manager is present
- Check for known vulnerable package versions
- Look for unnecessary dependencies that increase attack surface

## 5. Configuration
- Check for debug mode enabled in production configs
- Look for CORS misconfigurations (wildcard origins)
- Check for missing security headers (CSP, HSTS, X-Frame-Options)
- Verify TLS/SSL configuration if present

## Output Format
Provide a structured report with:
- **CRITICAL**: Immediate action required (secrets exposed, injection vectors)
- **HIGH**: Should fix before deployment (missing auth, vulnerable deps)
- **MEDIUM**: Best practice violations (missing headers, weak config)
- **LOW**: Recommendations (code quality, defense in depth)

For each finding, include: file path, line number, description, and recommended fix.
