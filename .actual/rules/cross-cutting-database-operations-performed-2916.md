# Use PostgreSQL Connection Pool for Database Access: Database Operations Performed

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PGPOOL-001** SHOULD: Database operations SHOULD be performed within async route handlers to properly handle connection lifecycle

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized syntax with $1, $2 placeholders
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'
```

**Accept when:**
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations
- Database operations are executed within async route handlers to maintain proper connection lifecycle

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting changes to database access patterns.
</enforcement>