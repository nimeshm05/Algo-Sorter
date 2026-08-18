# Adopt PostgreSQL with Parameterized Queries for Primary Data Access: Visitor Tracking Implement

These rules are ALWAYS ACTIVE for all database access code in the visitor tracking implementation, including route handlers, data access patterns, and connection pool initialization.

### Rules

- **R-PGSQL-001** MUST: Visitor tracking MUST implement three query patterns: SELECT SUM for aggregation, SELECT with WHERE for lookup, INSERT for new records, and UPDATE with WHERE for modifications.
- **R-PGSQL-002** MUST: All user-supplied data (IP addresses, timestamps) MUST be passed via parameterized queries using separate text and values properties in pool.query calls.
- **R-PGSQL-003** MUST: Pool instance MUST be initialized once at application startup using process.env.DATABASE_URL and reused across all route handlers.
- **R-PGSQL-004** MUST: Query results MUST be accessed via the rows array property with length validation before accessing indexed elements.
- **R-PGSQL-005** SHOULD: Direct query result property access (rows[0].sum, rows[0].count) SHOULD include null checks and optional chaining to prevent runtime errors.
- **R-PGSQL-006** SHOULD: Connection pool error event handlers and query retry logic with exponential backoff SHOULD be implemented for transient database connectivity issues.
- **R-PGSQL-007** MAY: Prepared statements with explicit statement names MAY be implemented as an optimization when query performance profiling identifies parameterized query parsing as a bottleneck.

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
- Query result access includes validation of array length before accessing indexed elements

<enforcement>
Clause Code MUST NOT skip or defer verification. All three verify commands MUST pass before code is accepted. Violations detected by CI pipeline checks MUST block pull requests and production deployments.
</enforcement>