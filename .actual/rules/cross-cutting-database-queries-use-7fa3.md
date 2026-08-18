# Use PostgreSQL Connection Pool for Database Access: Database Queries Use

These rules are ALWAYS ACTIVE for all database query code in the application, particularly files that execute SQL queries against PostgreSQL using the pg library's Pool instance.

### Rules

- **R-DB-001** MUST: All database queries MUST use parameterized queries with the $1, $2 placeholder syntax to prevent SQL injection.

### Verify

```bash
# Verify Pool instance is created with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all pool.query calls use parameterized syntax with $1, $2 placeholders
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ | grep -q 'false'
```

**Accept when:**
- A Pool instance is created with DATABASE_URL from environment variables and SSL configuration
- All database queries use parameterized syntax with $1, $2 placeholders for dynamic values
- The same pool instance is reused across multiple route handlers and query operations
- No direct string concatenation is used to construct SQL queries with user-provided data

<enforcement>
Claude Code MUST NOT skip or defer verification of parameterized query usage. All database queries must be inspected for SQL injection vulnerabilities before acceptance.
</enforcement>