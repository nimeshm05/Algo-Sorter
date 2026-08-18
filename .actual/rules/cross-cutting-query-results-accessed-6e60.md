# Adopt PostgreSQL with Parameterized Queries for Primary Data Access: Query Results Accessed

These rules are ALWAYS ACTIVE for all database query code in the Node.js Express application that accesses PostgreSQL via the pg library.

### Rules

- **R-PG-001** MUST: Use parameterized queries with separate `text` and `values` properties when executing any SQL statement that incorporates user-supplied data (IP addresses, timestamps, or request parameters).
- **R-PG-002** SHOULD: Access query results via the `rows` property of the query result object returned by `pool.query()`.
- **R-PG-003** MUST: Initialize the Pool instance once at application startup using `process.env.DATABASE_URL` and reuse it across all route handlers to maintain connection pooling efficiency.
- **R-PG-004** MUST: Validate array length and check for null/undefined values before accessing indexed elements or nested properties on query result rows.
- **R-PG-005** SHOULD: Use optional chaining or defensive null checks when accessing nested properties on result rows (e.g., `rows[0]?.sum`, `rows[0]?.count`).
- **R-PG-006** MUST: Never concatenate user-supplied data directly into SQL query strings; always use parameterized placeholders (`$1`, `$2`, etc.) with corresponding values array entries.

### Verify

```bash
# Check for unparameterized queries
grep -r 'pool\.query' server/src/ | grep -v '\$[0-9]' | grep -v 'SELECT SUM' && echo 'Found unparameterized queries' || echo 'All queries use parameters'

# Verify Pool uses environment configuration
grep -r 'new Pool' server/src/ | grep -q 'process\.env\.DATABASE_URL' && echo 'Pool uses env config' || echo 'Pool config not from env'

# Verify DML queries use parameterization
grep -r 'pool\.query' server/src/ | grep -E '(INSERT|UPDATE)' | grep -q 'values:' && echo 'DML queries use parameterization' || echo 'DML queries missing parameters'
```

**Accept when:**
- All grep commands for unparameterized queries return no matches, confirming parameterized query usage across SELECT, INSERT, and UPDATE operations
- Pool initialization verification confirms `DATABASE_URL` is sourced from `process.env`
- All INSERT and UPDATE statements include `values` arrays with parameterized placeholders (`$1`, `$2`, etc.)
- Query result access consistently uses the `rows` property with defensive null checks before accessing indexed elements
- No direct string concatenation of user-supplied data into SQL query text is present

<enforcement>
Claude Code MUST NOT skip or defer verification. All database query code must pass the parameterization and environment configuration checks before being considered compliant with this rule set.
</enforcement>