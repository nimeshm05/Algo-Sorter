# Use PostgreSQL Connection Pool for Database Access: Ssl Configuration Set

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PGPOOL-001** MUST: SSL configuration MUST set rejectUnauthorized to false for the connection pool

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized syntax with $1, $2 placeholders
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration sets rejectUnauthorized to false
grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'
```

**Accept when:**
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations
- SSL configuration explicitly sets rejectUnauthorized to false

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting code that modifies database connection patterns.
</enforcement>