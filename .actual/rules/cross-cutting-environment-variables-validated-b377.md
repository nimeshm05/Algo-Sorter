# Store Database Credentials in Environment Variables for PostgreSQL Connection Pooling: Environment Variables Validated

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the server application that handle database connections, configuration, and runtime initialization.

### Rules

- **R-ENV-001** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` and never hardcoded in source code.
- **R-ENV-002** MUST: Server port configuration MUST be sourced from `process.env.PORT` and never hardcoded in source code.
- **R-ENV-003** SHOULD: Environment variables SHOULD be validated at application startup to fail fast on misconfiguration.
- **R-ENV-004** MUST: No hardcoded database credentials or connection strings matching patterns like `postgresql://` or `postgres://` with embedded credentials MUST appear in source code.
- **R-ENV-005** SHOULD: A `.env.example` template file SHOULD be maintained documenting required environment variables without actual values.
- **R-ENV-006** SHOULD: Startup validation SHOULD check for required environment variables (DATABASE_URL, PORT) before initializing the connection pool.

### Verify

```bash
# Verify all database URLs are sourced from environment variables
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify port configuration uses environment variables
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded credentials exist in source code
! grep -rE '(postgresql://|postgres://).*:.*@' server/src/ --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL`
- Server port configuration is sourced from `process.env.PORT`
- No hardcoded database credentials or connection strings appear in source code
- Environment variable validation occurs at application startup
- A `.env.example` file documents required variables

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static code analysis for hardcoded credentials patterns is mandatory before accepting any changes to database connection or configuration code.
</enforcement>