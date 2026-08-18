# Use PostgreSQL with Connection Pooling for Data Access: Database Connections Enforce

These rules are ALWAYS ACTIVE for all Node.js/Express application code that performs database operations using PostgreSQL, particularly files initializing database connections and executing queries against the visitors table or any persistent data store.

### Rules

- **R-DB-001** MUST: All database connections MUST enforce SSL with rejectUnauthorized set to false for cloud database providers.
- **R-DB-002** MUST: Initialize the Pool instance once at module level and reuse it across all route handlers to maintain connection pooling benefits.
- **R-DB-003** MUST: Always use parameterized queries (text/values pattern) when incorporating user input, request parameters, or any external data into SQL statements.
- **R-DB-004** MUST: Wrap all pool.query() calls in try-catch blocks and implement appropriate error responses for database failures.
- **R-DB-005** SHOULD: Consider adding connection pool event listeners (pool.on('error')) to log connection issues and monitor pool health.
- **R-DB-006** SHOULD: Explicitly configure pool size, connection timeout, and idle timeout based on expected load and database connection limits.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'

# Verify parameterized queries are used
grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'

# Check for direct Client usage (should not exist)
grep -r 'new Client' server/src/ | grep -v 'Pool' && echo 'WARNING: Direct Client connections found'

# Check for SQL concatenation patterns (should not exist)
grep -r "query.*+.*process\.env\|query.*template.*literal" server/src/ && echo 'WARNING: Potential SQL injection patterns found'
```

**Accept when:**
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options
- All pool.query() calls are wrapped in try-catch blocks or promise error handlers
- No raw string concatenation or template literals are used to construct SQL queries

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-DB rules marked MUST are non-negotiable and must be verified before accepting code changes. Security-critical rules (R-DB-001, R-DB-003, R-DB-004) require explicit confirmation of compliance.
</enforcement>