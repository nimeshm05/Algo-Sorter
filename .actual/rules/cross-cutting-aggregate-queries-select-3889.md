# Use PostgreSQL with Connection Pooling for Data Access: Aggregate Queries Select

These rules are ALWAYS ACTIVE for all Node.js/Express application code that performs database operations against PostgreSQL, particularly code in `server/src/` that initializes database connections or executes queries.

### Rules

- **R-PG-001** MUST: Use the pg library's Pool class for connection management, initialized once at module level with DATABASE_URL from environment variables.
- **R-PG-002** MUST: All queries containing user-supplied data, request parameters, or external data MUST use parameterized queries with positional placeholders ($1, $2, $3, etc.).
- **R-PG-003** MUST: SSL configuration MUST be explicitly defined in Pool initialization options with appropriate certificate handling for the deployment environment.
- **R-PG-004** SHOULD: Aggregate queries (SELECT SUM) SHOULD be used for computing derived metrics across the dataset rather than fetching and aggregating in application code.
- **R-PG-005** SHOULD: All pool.query() calls SHOULD be wrapped in try-catch blocks with appropriate error responses for database failures.
- **R-PG-006** MAY: Connection pool event listeners (pool.on('error')) MAY be added to log connection issues and monitor pool health.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'

# Verify parameterized queries are used
grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'

# Verify no direct Client connections are used
! grep -r 'new Client' server/src/ | grep -v 'Pool' && echo 'No direct Client connections found'

# Verify no raw SQL concatenation patterns
! grep -r "query.*\+.*process\.env" server/src/ && echo 'No raw SQL concatenation detected'
```

**Accept when:**
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options
- Aggregate queries (SELECT SUM) are used for computing derived metrics across the dataset
- All pool.query() calls are wrapped in try-catch blocks or equivalent error handling
- No raw SQL string concatenation patterns are present in the codebase

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PG-00X rules marked MUST are non-negotiable and must pass verification before code is accepted. SHOULD rules represent best practices that should be followed unless explicitly documented exceptions exist with security review approval.
</enforcement>