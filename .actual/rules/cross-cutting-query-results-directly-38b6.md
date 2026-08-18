# Use PostgreSQL with Connection Pooling for Data Access: Query Results Directly

These rules are ALWAYS ACTIVE for all Node.js/Express application code that performs database operations against PostgreSQL using the pg library.

### Rules

- **R-PGPOOL-001** MUST: Initialize the Pool instance once at module level and reuse it across all route handlers to maintain connection pooling benefits.
- **R-PGPOOL-002** MUST: Always use parameterized queries (text/values pattern) when incorporating user input, request parameters, or any external data into SQL statements.
- **R-PGPOOL-003** MUST: Wrap all pool.query() calls in try-catch blocks and implement appropriate error responses for database failures.
- **R-PGPOOL-004** SHOULD: Configure pool size, connection timeout, and idle timeout explicitly based on expected load and database connection limits.
- **R-PGPOOL-005** SHOULD: Add connection pool event listeners (pool.on('error')) to log connection issues and monitor pool health.
- **R-PGPOOL-006** MAY: Query results MAY be directly accessed via the rows array property for result set processing.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'

# Verify parameterized queries are used
grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'
```

**Accept when:**
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options
- All pool.query() calls are wrapped in try-catch blocks or promise error handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PGPOOL rules must be validated before approving database access code. Violations of R-PGPOOL-001, R-PGPOOL-002, or R-PGPOOL-003 are blocking issues requiring remediation.
</enforcement>