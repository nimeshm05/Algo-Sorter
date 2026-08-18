# Use PostgreSQL with Connection Pooling for Data Access: Data Access Operations

These rules are ALWAYS ACTIVE for all data access code in the application that interacts with PostgreSQL, including route handlers, database utility modules, and any code executing SQL queries.

### Rules

- **R-DA-001** MUST: Initialize the Pool instance once at module level and reuse it across all route handlers to maintain connection pooling benefits.
- **R-DA-002** MUST: Always use parameterized queries (text/values pattern) when incorporating user input, request parameters, or any external data into SQL statements.
- **R-DA-003** MUST: Use positional placeholders ($1, $2, $3, etc.) for all parameterized queries to prevent SQL injection vulnerabilities.
- **R-DA-004** MUST: Wrap all pool.query() calls in try-catch blocks and implement appropriate error responses for database failures.
- **R-DA-005** SHOULD: Data access operations SHOULD use async/await patterns for query execution within route handlers.
- **R-DA-006** SHOULD: Configure SSL with explicit certificate handling; avoid rejectUnauthorized: false in production without documented justification and security review.
- **R-DA-007** SHOULD: Explicitly configure pool size, connection timeout, and idle timeout based on expected load and database connection limits.
- **R-DA-008** SHOULD: Add connection pool event listeners (pool.on('error')) to log connection issues and monitor pool health.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'

# Verify parameterized queries are used
grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'

# Verify no direct Client connections (anti-pattern)
! grep -r 'new Client' server/src/ | grep -v 'Pool' && echo 'No direct Client connections found'

# Verify try-catch wrapping around pool.query calls
grep -r 'pool.query' server/src/ | grep -B2 'try' && echo 'Try-catch blocks detected'
```

**Accept when:**
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options
- All pool.query() calls are wrapped in try-catch blocks with error handling
- Connection pool is initialized once at module level and reused across route handlers
- Pool event listeners are configured to monitor connection health

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All pull requests introducing data access code must pass verification commands and code review confirming adherence to parameterized queries and connection pooling patterns. Violations involving SQL concatenation or direct Client usage must be rejected immediately.
</enforcement>