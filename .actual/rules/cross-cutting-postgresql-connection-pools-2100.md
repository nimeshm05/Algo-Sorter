# Store Database Connection Strings in Environment Variables: Postgresql Connection Pools

These rules are ALWAYS ACTIVE for all server-side code that initializes database connections, particularly PostgreSQL Pool initialization and runtime configuration in `server/src/`.

### Rules

- **R-PSQL-001** MUST: PostgreSQL connection pools MUST be initialized with SSL configuration using rejectUnauthorized setting.
- **R-PSQL-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-PSQL-003** MUST: No hardcoded `postgresql://` connection strings containing credentials are permitted outside of environment variable references.
- **R-PSQL-004** SHOULD: Create `.env.example` file documenting required environment variables (DATABASE_URL, PORT) without actual credentials.
- **R-PSQL-005** SHOULD: Implement startup validation that checks for DATABASE_URL presence and attempts connection before accepting requests.

### Verify

```bash
# Check for process.env.DATABASE_URL usage in connection initialization
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify no hardcoded postgresql:// credentials exist outside environment variable references
grep -r 'postgresql://' server/src/ | grep -v 'process.env' | grep -v 'node_modules' && echo 'FAIL: Hardcoded credentials found' || echo 'PASS: No hardcoded credentials'

# Verify Pool initialization includes SSL configuration and environment variables
grep -r 'new Pool' server/src/ | grep -A 5 'connectionString' | grep 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files
- PostgreSQL Pool initialization includes SSL configuration and references environment variables
- No grep matches for hardcoded `postgresql://` connection strings outside of environment variable references
- `.env.example` file exists documenting required environment variables without actual credentials
- Startup validation checks for DATABASE_URL presence before accepting requests

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to database connection initialization code.
</enforcement>