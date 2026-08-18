# Externalize Database Credentials via Environment Variables: Database Connection Strings

These rules are ALWAYS ACTIVE for all server-side application code that initializes database connections or configures runtime parameters.

### Rules

- **R-DBCRED-001** MUST: Database connection strings MUST be sourced from `process.env.DATABASE_URL` rather than hardcoded in application code.
- **R-DBCRED-002** MUST: Server port binding MUST use `process.env.PORT` with no port numbers hardcoded in application initialization.
- **R-DBCRED-003** MUST: No credential strings, connection URLs, or sensitive connection parameters SHALL appear in version control history or current source files.
- **R-DBCRED-004** SHOULD: Create a `.env.example` file documenting all required environment variables with placeholder values for developer onboarding.
- **R-DBCRED-005** SHOULD: Implement startup validation that checks `process.env.DATABASE_URL` and `process.env.PORT` are defined before initializing the connection pool.
- **R-DBCRED-006** SHOULD: Add `.env` to `.gitignore` and provide pre-commit hooks to scan for credential patterns.

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
- `.env` file is present in `.gitignore`.
- Startup validation logic checks for required environment variables before connection pool initialization.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database connection initialization code MUST be inspected to confirm externalization of credentials via environment variables.
</enforcement>