# Externalize Database Credentials via Environment Variables: Connection Pool Initialization

These rules are ALWAYS ACTIVE for all database connection initialization code, server runtime configuration, and any code paths that consume environment variables for credentials or deployment-specific settings.

### Rules

- **R-CRED-001** SHOULD: Connection pool initialization SHOULD validate environment variables are present before establishing database connections.

### Verify

```bash
# Verify environment variable usage for database and port configuration
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Detect hardcoded credentials or connection strings in source code
grep -r 'postgresql://\|postgres://\|password.*=\|user.*=' server/src/ | grep -v process.env

# Verify .env is in .gitignore to prevent credential commits
test -f .gitignore && grep -q '\.env' .gitignore
```

**Accept when:**
- All database connection parameters are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files
- Server port binding uses `process.env.PORT` and no port numbers are hardcoded in application initialization
- No credential strings or connection URLs appear in version control history or current source files
- `.env` file is listed in `.gitignore` to prevent accidental credential commits
- Startup validation checks that required environment variables (`DATABASE_URL`, `PORT`) are defined before initializing the connection pool

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database connection initialization code MUST be reviewed to ensure credentials are externalized via environment variables and no hardcoded secrets exist in source files.
</enforcement>