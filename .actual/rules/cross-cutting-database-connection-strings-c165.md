# Adopt PostgreSQL with Parameterized Queries for Primary Data Access: Database Connection Strings

These rules are ALWAYS ACTIVE for all database access code in the application, particularly files performing SELECT, INSERT, and UPDATE operations against PostgreSQL via the pg library.

### Rules

- **R-DB-001** MUST: Database connection strings MUST be sourced from environment variables, never hardcoded in application code.
- **R-DB-002** MUST: All SQL queries incorporating user-supplied data (IP addresses, timestamps, request parameters) MUST use parameterized queries with separate text and values properties.
- **R-DB-003** MUST: The Pool instance MUST be initialized once at application startup and reused across all route handlers to maintain connection pooling efficiency.
- **R-DB-004** MUST: Query results MUST be validated for array length before accessing indexed elements to prevent runtime errors on unexpected query results.
- **R-DB-005** SHOULD: Connection pool error event handlers and query retry logic with exponential backoff SHOULD be implemented to handle transient database connectivity issues.
- **R-DB-006** SHOULD: SSL configuration for production database instances SHOULD set rejectUnauthorized: true or implement certificate pinning rather than accepting self-signed certificates.

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
- All grep commands for unparameterized queries return no matches, confirming parameterized query usage across SELECT, INSERT, and UPDATE operations.
- Pool initialization verification confirms DATABASE_URL is sourced from process.env.
- All INSERT and UPDATE statements include values arrays with parameterized placeholders ($1, $2, etc.).
- No hardcoded database credentials or connection strings are present in application code files.

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands MUST pass before accepting database access code changes. Violations detected by CI pipeline checks MUST block pull requests and production deployments.
</enforcement>