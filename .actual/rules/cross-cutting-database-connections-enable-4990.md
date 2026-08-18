# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Database Connections Enable

These rules are ALWAYS ACTIVE for all database connection initialization and query execution code in the application, particularly in `server/src/index.ts` and any database access modules.

### Rules

- **R-DB-001** MUST: Database connections MUST enable SSL with rejectUnauthorized set to false for managed database services.
- **R-DB-002** MUST: Initialize the Pool instance once at application startup with DATABASE_URL and SSL configuration.
- **R-DB-003** MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders to prevent SQL injection.
- **R-DB-004** SHOULD: Extract database access into a separate module (e.g., db.ts) to isolate pool configuration and query functions.
- **R-DB-005** SHOULD: Add connection pool event listeners (pool.on('error', ...)) to handle unexpected connection errors gracefully.
- **R-DB-006** SHOULD: Configure explicit pool connection limits based on database service tier and expected concurrent load.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL' && \
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized: false'

# Verify all database queries use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify no hardcoded connection strings
! grep -r 'postgresql://' server/src/ | grep -v 'DATABASE_URL'
```

**Accept when:**
- Pool initialization with DATABASE_URL and SSL configuration is present in server/src/index.ts
- All database queries use parameterized statements with $1, $2, etc. placeholders
- SSL configuration includes rejectUnauthorized: false setting for managed database compatibility
- No hardcoded database credentials or connection strings are present in source code

<enforcement>
Claude Code MUST NOT skip or defer verification of these database connection rules. All R-DB-### rules marked MUST are non-negotiable for security and reliability. Violations in parameterized query patterns or SSL configuration MUST block code acceptance.
</enforcement>