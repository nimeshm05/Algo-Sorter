# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Database Connection Configuration

These rules are ALWAYS ACTIVE for all database connection initialization and query execution code in the application, particularly in `server/src/index.ts` and any database access modules.

### Rules

- **R-DB-001** MUST: Database connection configuration MUST be isolated at application initialization before middleware registration.
- **R-DB-002** MUST: All database queries MUST use parameterized statements with `$1`, `$2`, etc. placeholders to prevent SQL injection.
- **R-DB-003** MUST: PostgreSQL Pool initialization MUST include SSL configuration with `rejectUnauthorized: false` for managed database service compatibility.
- **R-DB-004** MUST: Database credentials MUST be sourced from the `DATABASE_URL` environment variable and MUST NOT be hardcoded.
- **R-DB-005** SHOULD: Database access SHOULD be extracted into a separate module (e.g., `db.ts`) to isolate pool configuration and query functions.
- **R-DB-006** SHOULD: Connection pool event listeners SHOULD be registered (e.g., `pool.on('error', ...)`) to handle unexpected connection errors gracefully.
- **R-DB-007** MAY: Explicit pool configuration with max connection limits MAY be added based on database service tier and expected concurrent load.

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
- Pool initialization with `DATABASE_URL` and SSL configuration is present in `server/src/index.ts`
- All database queries use parameterized statements with `$1`, `$2`, etc. placeholders
- SSL configuration includes `rejectUnauthorized: false` for managed database compatibility
- No hardcoded database credentials or connection strings are present in source code
- Connection pool is initialized before middleware and route registration

<enforcement>
Claude Code MUST NOT skip or defer verification of these database connection rules. All R-DB rules marked MUST are non-negotiable and MUST be verified before accepting any database connection implementation.
</enforcement>