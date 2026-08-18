# Store Database Connection Strings in Environment Variables: Database Credentials Not

These rules are ALWAYS ACTIVE for all server-side code files that initialize database connections or reference database configuration.

### Rules

- **R-DB-001** MUST NOT: Database credentials MUST NOT be hardcoded in source code or committed to version control.
- **R-DB-002** MUST: All database connection strings MUST be sourced from environment variables (e.g., `process.env.DATABASE_URL`).
- **R-DB-003** MUST: PostgreSQL Pool initialization MUST reference environment variables for connection configuration.
- **R-DB-004** MUST: SSL/TLS configuration for database connections MUST be sourced from environment variables or configuration.
- **R-DB-005** SHOULD: Implement startup validation to check for required environment variables before accepting requests.
- **R-DB-006** SHOULD: Create `.env.example` file documenting required environment variables without actual credentials.

### Verify

```bash
# Check for process.env.DATABASE_URL usage in connection initialization
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify no hardcoded postgresql:// connection strings outside environment variable references
grep -r 'postgresql://' server/src/ | grep -v 'process.env' | grep -v 'node_modules' && echo 'FAIL: Hardcoded credentials found' || echo 'PASS: No hardcoded credentials'

# Verify PostgreSQL Pool initialization includes environment variable references
grep -r 'new Pool' server/src/ | grep -A 5 'connectionString' | grep 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- PostgreSQL Pool initialization includes SSL configuration and references environment variables.
- No grep matches for hardcoded `postgresql://` connection strings outside of environment variable references.
- Startup validation checks for required environment variables before accepting requests.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database connection code MUST be reviewed to ensure credentials are externalized to environment variables.
</enforcement>