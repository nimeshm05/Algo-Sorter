# Store Database Connection Strings in Environment Variables: Database Connection Strings

These rules are ALWAYS ACTIVE for all server-side code that initializes database connections or references database configuration.

### Rules

- **R-DB-001** MUST: Database connection strings MUST be sourced from process.env.DATABASE_URL environment variable

### Verify

```bash
# Check that DATABASE_URL is referenced from environment variables
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify no hardcoded postgresql:// connection strings exist outside environment variable references
grep -r 'postgresql://' server/src/ | grep -v 'process.env' | grep -v 'node_modules' && echo 'FAIL: Hardcoded credentials found' || echo 'PASS: No hardcoded credentials'

# Verify PostgreSQL Pool initialization includes environment variable reference
grep -r 'new Pool' server/src/ | grep -A 5 'connectionString' | grep 'process.env'
```

**Accept when:**
- All database connection strings are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- PostgreSQL Pool initialization includes SSL configuration and references environment variables
- No grep matches for hardcoded postgresql:// connection strings outside of environment variable references
- Test fixtures may use hardcoded test database URLs only for isolated unit tests with clearly marked test data
- Documentation examples use placeholder values (e.g., postgresql://user:pass@host:5432/db) not real credentials

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database connection configuration must be validated against process.env.DATABASE_URL sourcing before code is approved.
</enforcement>