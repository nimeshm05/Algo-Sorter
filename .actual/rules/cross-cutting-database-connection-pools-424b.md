# Store Database Credentials in Environment Variables for PostgreSQL Connection Pooling: Database Connection Pools

These rules are ALWAYS ACTIVE for all server-side code that initializes database connections, manages connection pools, or configures runtime environment-specific values for PostgreSQL and Express applications.

### Rules

- **R-DB-001** SHOULD: Database connection pools SHOULD be initialized once at application startup and reused across requests.
- **R-DB-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL`.
- **R-DB-003** MUST: Server port configuration MUST be sourced from `process.env.PORT`.
- **R-DB-004** MUST: No hardcoded database credentials or connection strings (matching patterns like `postgresql://` or `postgres://` with embedded credentials) MUST appear in source code.
- **R-DB-005** SHOULD: A `.env.example` template file SHOULD be created documenting required environment variables without actual values.
- **R-DB-006** SHOULD: Startup validation SHOULD check for required environment variables (`DATABASE_URL`, `PORT`) and fail fast with clear error messages before initializing the connection pool.
- **R-DB-007** SHOULD: Logging and error handling SHOULD implement filters to redact sensitive environment variable values.

### Verify

```bash
# Verify all database connection strings are sourced from process.env.DATABASE_URL
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify server port configuration is sourced from process.env.PORT
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded database credentials or connection strings appear in source code
! grep -rE '(postgresql://|postgres://).*:.*@' server/src/ --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL`
- Server port configuration is sourced from `process.env.PORT`
- No hardcoded database credentials or connection strings appear in source code
- Startup validation checks for required environment variables before initializing the connection pool
- Connection pool is initialized once at application startup and reused across requests

<enforcement>
Claude Code MUST NOT skip or defer verification. Static code analysis, code review, and pre-commit hooks MUST detect violations. CI pipeline MUST fail on detection of hardcoded credentials. Security team MUST be notified of any credential exposure incidents.
</enforcement>