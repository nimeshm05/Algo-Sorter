# Adopt PostgreSQL with Parameterized Queries for Primary Data Access: Timestamp Fields Use

These rules are ALWAYS ACTIVE for all database access code in the Node.js Express application, particularly for visitor tracking data persistence operations involving INSERT, UPDATE, and SELECT queries against PostgreSQL.

### Rules

- **R-PGSQL-001** MUST: Use parameterized queries with separate `text` and `values` properties when incorporating user-supplied data (IP addresses, timestamps) to prevent SQL injection attacks.
- **R-PGSQL-002** MUST: Initialize the Pool instance once at application startup and reuse across all route handlers to maintain connection pooling efficiency.
- **R-PGSQL-003** MUST: Source DATABASE_URL from environment variables (process.env.DATABASE_URL) following twelve-factor app principles; never hardcode database credentials.
- **R-PGSQL-004** SHOULD: Timestamp fields SHOULD use `new Date()` for current timestamp values in INSERT and UPDATE operations.
- **R-PGSQL-005** MUST: Access query results via the `rows` array property and validate array length before accessing indexed elements to prevent runtime errors.
- **R-PGSQL-006** SHOULD: Add defensive null checks and result validation before accessing nested properties; consider using optional chaining for safe property access.
- **R-PGSQL-007** MUST: Structure all INSERT and UPDATE statements with values arrays containing parameterized placeholders ($1, $2, etc.) rather than string concatenation.

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
- Pool initialization verification confirms DATABASE_URL is sourced from process.env
- All INSERT and UPDATE statements include values arrays with parameterized placeholders ($1, $2, etc.)
- Query results are accessed defensively with array length validation before indexed element access
- No direct string concatenation or template literals are used to construct SQL query text with user-supplied data

<enforcement>
Claude Code MUST NOT skip or defer verification. All database access code modifications require passing all verify commands before acceptance. Violations detected by CI pipeline checks must be resolved before merge.
</enforcement>