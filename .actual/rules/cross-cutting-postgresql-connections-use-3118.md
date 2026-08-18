# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Postgresql Connections Use

These rules are ALWAYS ACTIVE for all Node.js/TypeScript applications using PostgreSQL as the primary datastore, including connection initialization, query execution, and database operations triggered by HTTP request handlers or background jobs.

### Rules

- **R-PGPOOL-001** MUST: PostgreSQL connections MUST use connection pooling via the pg Pool class to prevent connection exhaustion.
- **R-PGPOOL-002** MUST: All PostgreSQL Pool instantiations MUST include an ssl configuration object with an explicit rejectUnauthorized setting.
- **R-PGPOOL-003** MUST: All database queries with dynamic values MUST use parameterized queries (text and values properties) with no string concatenation.
- **R-PGPOOL-004** MUST: DATABASE_URL MUST be sourced from process.env and MUST NOT be hardcoded in any source files.
- **R-PGPOOL-005** MUST: Pool instances MUST be initialized once at application startup and reused across all request handlers to avoid creating multiple pools.
- **R-PGPOOL-006** MUST: Applications MUST implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-PGPOOL-007** SHOULD: Pool size SHOULD be configured using environment variables (e.g., PGMAXCONNECTIONS) to tune for specific deployment environments and database limits.
- **R-PGPOOL-008** SHOULD: For managed database services requiring rejectUnauthorized: false, inline comments SHOULD explain the hosting provider's certificate configuration.

### Verify

```bash
# Verify all PostgreSQL Pool instantiations include ssl configuration
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Verify parameterized query usage (count queries with $N placeholders)
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from environment, not hardcoded
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Verify no hardcoded connection strings
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- Pool instances are initialized once at application startup and reused across request handlers
- Graceful shutdown logic calls pool.end() before process termination

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PGPOOL rules are mandatory for PostgreSQL database access patterns. Violations MUST be flagged during code review and CI pipeline checks MUST fail if Pool instantiation lacks SSL configuration or queries use string concatenation instead of parameterized values.
</enforcement>