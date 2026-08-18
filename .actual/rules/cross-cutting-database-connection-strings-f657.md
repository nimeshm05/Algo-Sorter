# Store Database Credentials in Environment Variables for PostgreSQL Connection Pooling: Database Connection Strings

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the server application that handle database connections, configuration, and environment setup.

### Rules

- **R-CRED-001** MUST: Database connection strings MUST be sourced from `process.env.DATABASE_URL` environment variable.
- **R-CRED-002** MUST: Server port configuration MUST be sourced from `process.env.PORT` environment variable.
- **R-CRED-003** MUST: No hardcoded database credentials or connection strings (matching patterns like `postgresql://` or `postgres://` with embedded credentials) SHALL appear in source code.
- **R-CRED-004** SHOULD: Implement startup validation that checks for required environment variables (`DATABASE_URL`, `PORT`) before initializing the connection pool and fails fast with clear error messages.
- **R-CRED-005** SHOULD: Create and maintain a `.env.example` template file documenting required environment variables without actual values.
- **R-CRED-006** SHOULD: Implement logging filters to redact sensitive environment variable values from logs and error messages.

### Verify

```bash
# Verify DATABASE_URL is being sourced from environment
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify PORT is being sourced from environment
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded credentials appear in source code
! grep -rE '(postgresql://|postgres://).*:.*@' server/src/ --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL`
- Server port configuration is sourced from `process.env.PORT`
- No hardcoded database credentials or connection strings appear in source code
- Startup validation is implemented to check for required environment variables
- `.env.example` template file exists documenting required variables

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. Static code analysis, code review, and pre-commit hooks MUST detect violations. CI pipeline MUST fail on detection of hardcoded credentials.
</enforcement>