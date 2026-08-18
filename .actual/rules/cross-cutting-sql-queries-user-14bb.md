# Adopt PostgreSQL with Parameterized Queries for Primary Data Access: Sql Queries User

These rules are ALWAYS ACTIVE for all SQL query code in the application that accesses visitor tracking data, including SELECT, INSERT, and UPDATE operations against PostgreSQL.

### Rules

- **R-SQL-001** MUST: All SQL queries with user-supplied data MUST use parameterized queries with $1, $2, etc. placeholders and separate values arrays.
- **R-SQL-002** MUST: Initialize the Pool instance once at application startup and reuse across all route handlers to maintain connection pooling efficiency.
- **R-SQL-003** MUST: Always structure queries with separate text and values properties when incorporating user-supplied data (req.ip, timestamps).
- **R-SQL-004** MUST: Access query results via the rows array property and validate array length before accessing indexed elements.
- **R-SQL-005** MUST: Store DATABASE_URL in environment-specific configuration files (.env for development) and secure secret management systems for production.
- **R-SQL-006** SHOULD: Add defensive null checks and result validation before accessing nested properties; consider using optional chaining.
- **R-SQL-007** SHOULD: Implement connection pool error event handlers and query retry logic with exponential backoff for transient database connectivity issues.

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
- All grep commands for unparameterized queries return no matches, confirming parameterized query usage
- Pool initialization verification confirms DATABASE_URL is sourced from process.env
- All INSERT and UPDATE statements include values arrays with parameterized placeholders
- No direct string concatenation or template literals are used in SQL query construction
- Query results are validated for array length and null checks before property access

<enforcement>
Claude Code MUST NOT skip or defer verification. All SQL query code MUST be reviewed against R-SQL-001 through R-SQL-007. Violations of R-SQL-001 through R-SQL-005 are blocking issues. Violations of R-SQL-006 and R-SQL-007 should be flagged for remediation in follow-up work.
</enforcement>