# Externalize Database Credentials via Environment Variables: Credentials Connection Strings

These rules are ALWAYS ACTIVE for all application code, configuration files, and deployment artifacts that handle database connectivity, server runtime configuration, or environment-specific settings.

### Rules

- **R-CRED-001** MUST NOT: Credentials, connection strings, or other secrets MUST NOT be committed to version control in plaintext.
- **R-CRED-002** MUST: All database connection parameters MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-CRED-003** MUST: Server port binding MUST use `process.env.PORT` and no port numbers MUST be hardcoded in application initialization.
- **R-CRED-004** MUST: Startup validation MUST check that `process.env.DATABASE_URL` and `process.env.PORT` are defined before initializing the connection pool.
- **R-CRED-005** MUST: `.env` files MUST be added to `.gitignore` to prevent accidental credential commits.
- **R-CRED-006** SHOULD: A `.env.example` file SHOULD be created documenting all required environment variables with placeholder values for developer onboarding.
- **R-CRED-007** SHOULD: Logging filters SHOULD be implemented to redact environment variable values and avoid echoing configuration in error responses.

### Verify

```bash
# Verify environment variable usage for DATABASE_URL and PORT
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Detect hardcoded credentials or connection strings
grep -r 'postgresql://\|postgres://\|password.*=\|user.*=' server/src/ | grep -v process.env

# Verify .env is gitignored
test -f .gitignore && grep -q '\.env' .gitignore
```

**Accept when:**
- All database connection parameters are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- Server port binding uses `process.env.PORT` and no port numbers are hardcoded in application initialization.
- No credential strings or connection URLs appear in version control history or current source files.
- `.env` is present in `.gitignore`.
- Startup validation logic checks for required environment variables and fails fast with clear error messages.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-CRED rules marked MUST are non-negotiable and must be verified before accepting code changes. Violations require immediate remediation and credential rotation if secrets were exposed.
</enforcement>