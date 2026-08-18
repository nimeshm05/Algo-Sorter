# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Applications Expose Runtime

These rules are ALWAYS ACTIVE for all Node.js/TypeScript server applications using PostgreSQL as the primary datastore, particularly those handling concurrent HTTP requests with visitor tracking and stateful data persistence operations.

### Rules

- **R-POOL-001** MUST: Initialize PostgreSQL Pool instances at module level before defining route handlers to ensure single pool reuse across all database operations.
- **R-POOL-002** MUST: Source database connection credentials from runtime environment variables (process.env.DATABASE_URL) rather than hardcoded configuration files.
- **R-POOL-003** MUST: Configure SSL for all PostgreSQL connections with explicit rejectUnauthorized setting in Pool initialization.
- **R-POOL-004** MUST: Use parameterized queries with separate text and values properties for all database operations to prevent SQL injection vulnerabilities.
- **R-POOL-005** MUST: Configure explicit pool size limits using max and min parameters based on expected concurrent request volume and database connection limits.
- **R-POOL-006** SHOULD: Implement graceful shutdown handling to close the pool on application termination using pool.end() in process signal handlers.
- **R-POOL-007** SHOULD: Add startup validation to verify DATABASE_URL is set and test database connectivity before binding to PORT to fail fast on misconfiguration.
- **R-POOL-008** MAY: Applications MAY expose runtime configuration through environment variables including PORT for service binding.

### Verify

```bash
# Verify Pool instances use connectionString from process.env.DATABASE_URL
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration is present in Pool initialization
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- Pool is initialized at module level before route handler definitions
- Explicit pool size limits (max and min parameters) are configured
- Startup validation checks for required environment variables before attempting database connection
- Graceful shutdown handling is implemented for pool.end() on process termination

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-POOL-001 through R-POOL-008 requirements MUST be validated before accepting code changes affecting PostgreSQL connection management.
</enforcement>