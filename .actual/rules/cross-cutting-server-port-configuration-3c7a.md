# Externalize Database Credentials via Environment Variables: Server Port Configuration

These rules are ALWAYS ACTIVE for all server initialization code, database connection configuration, and runtime environment setup across all deployment contexts.

### Rules

- **R-CRED-001** MUST: Server port configuration MUST be sourced from `process.env.PORT` to support platform-assigned port binding.
- **R-CRED-002** MUST: All database connection parameters MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-CRED-003** MUST: No credential strings, connection URLs, passwords, or usernames SHALL appear in version control history or current source files.
- **R-CRED-004** SHOULD: Implement startup validation that checks `process.env.DATABASE_URL` and `process.env.PORT` are defined before initializing the connection pool.
- **R-CRED-005** SHOULD: Create a `.env.example` file documenting all required environment variables with placeholder values for developer onboarding.
- **R-CRED-006** MAY: Local development environments may use `.env` files with non-production credentials for convenience, provided `.env` is in `.gitignore`.

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
- `.env` file is present in `.gitignore` to prevent accidental credential commits.
- Startup validation logic checks for required environment variables and fails fast with clear error messages.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All three verify commands MUST execute successfully before accepting changes to server initialization, database configuration, or environment variable handling.
</enforcement>