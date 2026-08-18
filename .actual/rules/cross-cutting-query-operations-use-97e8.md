# Use PostgreSQL Connection Pool for Database Access: Query Operations Use

These rules are ALWAYS ACTIVE for all database query operations and connection management code in the application.

### Rules

- **R-POOL-001** SHOULD: Query operations SHOULD use the pool.query() method with text and values properties for parameterized queries

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all queries use parameterized syntax with $1, $2 placeholders
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'
```

**Accept when:**
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All query operations must be reviewed for compliance with parameterized query patterns and connection pool reuse before code acceptance.
</enforcement>