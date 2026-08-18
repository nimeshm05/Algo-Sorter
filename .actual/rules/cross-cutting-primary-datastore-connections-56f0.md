# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Primary Datastore Connections

These rules are ALWAYS ACTIVE for all Node.js/TypeScript files in `server/src/` that initialize or use PostgreSQL database connections.

### Rules

- **R-PDS-001** MUST: Primary datastore connections MUST use the pg Pool class initialized with connectionString from process.env.DATABASE_URL
- **R-PDS-002** MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders to prevent SQL injection
- **R-PDS-003** MUST: SSL configuration MUST include rejectUnauthorized setting for managed database compatibility
- **R-PDS-004** SHOULD: Pool instance SHOULD be initialized once at application startup in server/src/index.ts before registering middleware or routes
- **R-PDS-005** SHOULD: Database access SHOULD be extracted into a separate module (e.g., db.ts) to isolate pool configuration and query functions
- **R-PDS-006** SHOULD: Connection pool event listeners SHOULD be added (pool.on('error', ...)) to handle unexpected connection errors gracefully

### Verify

```bash
# Verify Pool initialization with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration includes rejectUnauthorized setting
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized: false'
```

**Accept when:**
- Pool initialization with DATABASE_URL and SSL configuration is present in server/src/index.ts
- All database queries use parameterized statements with $1, $2, etc. placeholders
- SSL configuration includes rejectUnauthorized setting for managed database compatibility
- No hardcoded database credentials or connection strings are present in source code
- Connection pool event listeners are configured to handle errors gracefully

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-PDS rules marked MUST are non-negotiable and must pass verification before code is accepted. SHOULD rules represent best practices that should be followed unless explicitly documented exceptions are approved by the architecture team.
</enforcement>