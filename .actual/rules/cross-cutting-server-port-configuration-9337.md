# Store Database Credentials in Environment Variables for PostgreSQL Connection Pooling: Server Port Configuration

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript files in the server application that handle database connections, environment configuration, and server initialization.

### Rules

- **R-CRED-001** MUST: Server port configuration MUST be sourced from `process.env.PORT` environment variable.
- **R-CRED-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` environment variable.
- **R-CRED-003** MUST: No hardcoded database credentials or connection strings (matching patterns like `postgresql://` or `postgres://` with embedded credentials) shall appear in source code.
- **R-CRED-004** SHOULD: Implement startup validation that checks for required environment variables (`DATABASE_URL`, `PORT`) before initializing the connection pool or server.
- **R-CRED-005** SHOULD: Create and maintain a `.env.example` template file documenting required environment variables without actual values.

### Verify

```bash
# Verify DATABASE_URL is sourced from environment
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify PORT is sourced from environment
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded credentials in source code
! grep -rE '(postgresql://|postgres://).*:.*@' server/src/ --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL`
- Server port configuration is sourced from `process.env.PORT`
- No hardcoded database credentials or connection strings appear in source code
- Startup validation checks for required environment variables before initialization
- A `.env.example` template file exists documenting required variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and must be checked before accepting changes to database connection or server configuration code.
</enforcement>