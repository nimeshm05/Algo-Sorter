# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Pool Queries Use

These rules are ALWAYS ACTIVE for all database query code in the application's route handlers and data access layer.

### Rules

- **R-POOL-001** SHOULD: Pool queries SHOULD use the object form with text and values properties for parameterized queries.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration includes rejectUnauthorized setting
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized: false'
```

**Accept when:**
- Pool initialization with DATABASE_URL and SSL configuration is present in server/src/index.ts
- All database queries use parameterized statements with $1, $2, etc. placeholders
- SSL configuration includes rejectUnauthorized setting for managed database compatibility

<enforcement>
Claude Code MUST NOT skip or defer verification. All pool queries MUST be inspected for parameterized query patterns before approving changes to database access code.
</enforcement>