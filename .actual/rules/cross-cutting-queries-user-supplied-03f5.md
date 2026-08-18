# Use PostgreSQL with Connection Pooling for Data Access: Queries User Supplied

These rules are ALWAYS ACTIVE for all database access code in Node.js/Express applications using the pg library for PostgreSQL connections.

### Rules

- **R-PGPOOL-001** MUST: All queries with user-supplied data MUST use parameterized queries with the text/values pattern to prevent SQL injection

### Verify

```bash
# Verify Pool initialization with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'

# Verify parameterized queries are in use
grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'
```

**Accept when:**
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options

<enforcement>
Clause Code MUST NOT skip or defer verification. All pull requests introducing direct Client usage or non-parameterized queries must be rejected. Security scanning tools must flag SQL concatenation patterns for immediate remediation. Production deployments require passing integration tests that validate connection pool behavior.
</enforcement>