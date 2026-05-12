# Security

## Personal-Infra Posture

For personal infra: treat security as **paramount**. The user is not a security professional and relies on me to flag unsafe defaults. Bias to defense-in-depth even at convenience cost.

- Default to **private** repos for personal projects, even when no secrets are present (dotfiles still leak metadata).
- Never recommend `curl ... | bash` install patterns; clone-then-review.
- Strong key practices: ed25519 SSH keys with passphrases, **per-machine** keys (so revocation is granular), commit signing.
- Pre-commit secret scanning (e.g. `gitleaks`) on personal repos.
- Apply Keychain ACLs (`security ... -T ""`) for high-sensitivity items so each access prompts.
- Be explicit about threat models — never claim something is "secure" without naming the assumptions. When introducing an external service, lay out the threat model honestly (content sensitivity, accidental-leak vectors) and recommend the option that minimizes blast radius. Be explicit when an "identifier" is functionally a credential.

## Code-Level Rules

- Never hardcode secrets — use environment variables or Keychain
- Validate all external inputs at system boundaries
- Use parameterized queries; never interpolate user input into SQL
- No sensitive data in URL parameters or query strings

## Pre-Commit Checklist

- No hardcoded secrets (API keys, tokens, passwords)
- All user inputs validated at system boundaries
- Parameterized queries (no string concatenation in SQL)
- XSS prevention (sanitized HTML output)
- Errors don't leak sensitive data

If a security issue is found: stop, use `security-reviewer`, fix before continuing, rotate any exposed secrets.
