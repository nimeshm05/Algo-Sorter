# Store Database Credentials in Environment Variables for PostgreSQL Connection Pooling: Database Credentials Not

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the server application that handle database connections, configuration, and credential management.

### Rules

- **R-CRED-001** MUST_NOT: Database credentials MUST_NOT be hardcoded in source code or committed to version control.
- **R-CRED-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL`.
- **R-CRED-003** MUST: Server port configuration MUST be sourced from `process.env.PORT`.
- **R-CRED-004** MUST: Startup validation MUST check for required environment variables (DATABASE_URL, PORT) before initializing the connection pool and fail fast with clear error messages.
- **R-CRED-005** SHOULD: Create and maintain a `.env.example` template file documenting required environment variables without actual values.
- **R-CRED-006** SHOULD: Implement logging filters to redact sensitive environment variable values from logs and error messages.

### Verify

```bash
# Verify all database connection strings use process.env.DATABASE_URL
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify server port configuration uses process.env.PORT
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded database credentials or connection strings exist
! grep -rE '(postgresql://|postgres://).*:.*@' server/src/ --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL`
- Server port configuration is sourced from `process.env.PORT`
- No hardcoded database credentials or connection strings appear in source code
- Startup validation checks for required environment variables before connection pool initialization
- No credential patterns (postgresql://, postgres://) with embedded credentials are found outside of environment variable references

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Static code analysis and verification commands MUST be executed before accepting any changes to database connection or configuration code.
</enforcement>