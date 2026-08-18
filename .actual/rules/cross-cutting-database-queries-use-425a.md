# Use PostgreSQL Connection Pooling with SSL for Primary Datastore Access: Database Queries Use

These rules are ALWAYS ACTIVE for all PostgreSQL database connections in Node.js/TypeScript server applications, including visitor tracking and stateful data persistence operations in Express.js route handlers and cloud-hosted or containerized deployments requiring SSL database connections.

### Rules

- **R-DB-001** MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders and separate values arrays to prevent SQL injection vulnerabilities.
- **R-DB-002** MUST: All PostgreSQL connections MUST use Pool instances with connectionString sourced from process.env.DATABASE_URL for environment portability.
- **R-DB-003** MUST: Pool initialization MUST include SSL configuration with rejectUnauthorized explicitly set to accommodate cloud database providers while maintaining encrypted transport.
- **R-DB-004** MUST: The Pool instance MUST be initialized at module level before defining route handlers to ensure single pool reuse across concurrent requests.
- **R-DB-005** SHOULD: Pool size limits SHOULD be configured explicitly using max and min parameters based on expected concurrent request volume and database connection limits.
- **R-DB-006** SHOULD: Graceful shutdown handling SHOULD be implemented to close the pool on application termination using pool.end() in process signal handlers.
- **R-DB-007** SHOULD: Startup validation SHOULD verify DATABASE_URL is set and test database connectivity before binding to PORT to fail fast on misconfiguration.

### Verify

```bash
# Verify Pool instances use connectionString from process.env.DATABASE_URL
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'connectionString.*process.env.DATABASE_URL'

# Verify all database queries use parameterized statements with text and values properties
grep -r 'pool.query' server/src --include='*.ts' | grep -q 'text:.*values:'

# Verify SSL configuration is present in Pool initialization
grep -r 'new Pool' server/src --include='*.ts' | grep -q 'ssl.*rejectUnauthorized'
```

**Accept when:**
- All PostgreSQL connections use Pool instances with connectionString sourced from process.env.DATABASE_URL
- All database queries use parameterized statements with separate text and values properties
- SSL configuration is present in Pool initialization with rejectUnauthorized explicitly set
- Pool instance is initialized at module level before route handler definitions
- No direct pg.Client instances are used for database operations

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database query patterns MUST conform to parameterized statement requirements, and all Pool configurations MUST include SSL and environment-based connection strings before code is accepted.
</enforcement>