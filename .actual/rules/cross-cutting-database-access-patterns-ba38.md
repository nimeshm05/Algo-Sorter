# Use PostgreSQL Connection Pool with SSL for Primary Datastore Access: Database Access Patterns

These rules are ALWAYS ACTIVE for all database access code in the application, particularly server/src/index.ts and any modules handling PostgreSQL connections for visitor tracking operations.

### Rules

- **R-DB-001** MUST: Database access patterns MUST be limited to SELECT, INSERT, and UPDATE operations as defined in the visitor tracking schema.
- **R-DB-002** MUST: Pool initialization with DATABASE_URL and SSL configuration MUST be present in server/src/index.ts.
- **R-DB-003** MUST: All database queries MUST use parameterized statements with $1, $2, etc. placeholders to prevent SQL injection.
- **R-DB-004** MUST: SSL configuration MUST include rejectUnauthorized setting for managed database compatibility.
- **R-DB-005** SHOULD: Database access SHOULD be extracted into a separate module (e.g., db.ts) to isolate pool configuration and query functions.
- **R-DB-006** SHOULD: Connection pool event listeners (pool.on('error', ...)) SHOULD be added to handle unexpected connection errors gracefully.
- **R-DB-007** MAY: Pool configuration MAY include explicit max connection limits based on database service tier and expected concurrent load.

### Verify

```bash
# Verify Pool initialization with DATABASE_URL and SSL configuration
grep -r 'new Pool' server/src/ | grep -q 'connectionString.*DATABASE_URL'

# Verify all database queries use parameterized statements
grep -r 'pool\.query' server/src/ | grep -q '\$1'

# Verify SSL configuration with rejectUnauthorized setting
grep -r 'ssl:' server/src/ | grep -q 'rejectUnauthorized: false'
```

**Accept when:**
- Pool initialization with DATABASE_URL and SSL configuration is present in server/src/index.ts
- All database queries use parameterized statements with $1, $2, etc. placeholders
- SSL configuration includes rejectUnauthorized setting for managed database compatibility
- No DELETE or DROP operations are present in database query patterns
- Error handling does not log connection strings or database credentials

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database access patterns MUST conform to R-DB-001 through R-DB-004 before code review approval. Violations of parameterized query requirements (R-DB-003) MUST trigger CI pipeline failure.
</enforcement>