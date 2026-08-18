# Externalize Database Credentials via Environment Variables: Environment Variable Access

These rules are ALWAYS ACTIVE for all database connection configuration, server runtime configuration, and any configuration value that differs between development, staging, and production environments.

### Rules

- **R-ENV-001** SHOULD: Environment variable access SHOULD be centralized at application initialization to fail fast on missing configuration.

### Verify

```bash
# Verify environment variable usage for database and port configuration
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Detect hardcoded credentials in source files
grep -r 'postgresql://\|postgres://\|password.*=\|user.*=' server/src/ | grep -v process.env

# Verify .env is gitignored
test -f .gitignore && grep -q '\.env' .gitignore
```

**Accept when:**
- All database connection parameters are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- Server port binding uses process.env.PORT and no port numbers are hardcoded in application initialization
- No credential strings or connection URLs appear in version control history or current source files
- A .env.example file exists documenting all required environment variables with placeholder values
- Startup validation checks that process.env.DATABASE_URL and process.env.PORT are defined before initializing the connection pool

<enforcement>
Claude Code MUST NOT skip or defer verification of environment variable externalization. All database credentials and runtime configuration MUST be sourced from environment variables at application initialization, with no hardcoded secrets in source code.
</enforcement>