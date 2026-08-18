# Store Database Connection Strings in Environment Variables: Server Port Configuration

These rules are ALWAYS ACTIVE for all server-side code files that initialize database connections or configure runtime server ports.

### Rules

- **R-ENV-001** SHOULD: Server port configuration SHOULD be sourced from process.env.PORT environment variable
- **R-ENV-002** MUST: Database connection strings MUST NOT be hardcoded in source files
- **R-ENV-003** MUST: PostgreSQL Pool initialization MUST reference process.env.DATABASE_URL with no hardcoded credentials
- **R-ENV-004** SHOULD: SSL/TLS configuration SHOULD be sourced from environment variables and applied consistently across all environments

### Verify

```bash
# Check that process.env.DATABASE_URL is used for database connections
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify no hardcoded postgresql:// connection strings exist outside environment variable references
grep -r 'postgresql://' server/src/ | grep -v 'process.env' | grep -v 'node_modules' && echo 'FAIL: Hardcoded credentials found' || echo 'PASS: No hardcoded credentials'

# Confirm PostgreSQL Pool initialization includes environment variable references
grep -r 'new Pool' server/src/ | grep -A 5 'connectionString' | grep 'process.env'
```

**Accept when:**
- All database connection strings are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- PostgreSQL Pool initialization includes SSL configuration and references environment variables
- No grep matches for hardcoded postgresql:// connection strings outside of environment variable references
- Server port configuration references process.env.PORT without hardcoded port numbers in production code

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to database connection or server port configuration.
</enforcement>