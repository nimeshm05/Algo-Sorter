# Use PostgreSQL with Connection Pooling for Data Access: Database Connections Managed

These rules are ALWAYS ACTIVE for all Node.js/Express application code that performs database operations, particularly files in `server/src/` that initialize or use database connections.

### Rules

- **R-DB-001** MUST: Database connections MUST be managed through a connection pool initialized with connectionString from process.env.DATABASE_URL
- **R-DB-002** MUST: All database queries containing user-supplied data MUST use parameterized queries with positional placeholders ($1, $2, etc.)
- **R-DB-003** MUST: SSL configuration MUST be explicitly defined in Pool initialization options
- **R-DB-004** MUST: The Pool instance MUST be initialized once at module level and reused across all route handlers
- **R-DB-005** SHOULD: All pool.query() calls SHOULD be wrapped in try-catch blocks with appropriate error responses for database failures
- **R-DB-006** SHOULD: Connection pool event listeners (pool.on('error')) SHOULD be added to log connection issues and monitor pool health

### Verify

```bash
# Verify Pool initialization with DATABASE_URL
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && echo 'Pool initialization found'

# Verify parameterized queries are used
grep -r 'pool.query' server/src/ | grep -E '\$[0-9]' && echo 'Parameterized queries detected'

# Verify SSL configuration is present
grep -r 'ssl.*rejectUnauthorized' server/src/ && echo 'SSL configuration present'

# Check for direct Client usage (should not exist)
grep -r 'new Client' server/src/ && echo 'WARNING: Direct Client connections found' || echo 'No direct Client connections'

# Check for SQL string concatenation (should not exist)
grep -r "query.*+.*process\.env\|query.*template.*literal" server/src/ && echo 'WARNING: Potential SQL injection patterns found' || echo 'No obvious SQL concatenation patterns'
```

**Accept when:**
- All database queries use the Pool instance rather than direct Client connections
- All queries containing user-supplied data use parameterized queries with positional placeholders ($1, $2, etc.)
- SSL configuration is explicitly defined in Pool initialization options
- Pool is initialized once at module level and reused across route handlers
- No SQL string concatenation or template literals are used for query construction with external data

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database access patterns must be reviewed for compliance with R-DB-001 through R-DB-006 before code is accepted.
</enforcement>