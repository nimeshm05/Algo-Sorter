# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Parameterized Queries Used

These rules are ALWAYS ACTIVE for all Node.js/TypeScript files that initialize PostgreSQL connections, execute database queries, or manage connection pooling in the primary datastore access layer.

### Rules

- **R-PGPOOL-001** MUST: Parameterized queries MUST be used for all database operations involving user input or dynamic values to prevent SQL injection.
- **R-PGPOOL-002** MUST: All PostgreSQL Pool instantiations MUST include an ssl configuration object with an explicit rejectUnauthorized setting.
- **R-PGPOOL-003** MUST: DATABASE_URL MUST be sourced from process.env and MUST NOT be hardcoded in any source files.
- **R-PGPOOL-004** MUST: Pool instances MUST be initialized once at application startup and reused across all request handlers to avoid creating multiple pools.
- **R-PGPOOL-005** MUST: Graceful shutdown MUST call pool.end() to drain connections before process termination.

### Verify

```bash
# Verify Pool instantiation includes SSL configuration with explicit rejectUnauthorized setting
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Count parameterized queries using text/values pattern
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from process.env
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Detect hardcoded connection strings or credentials
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v process.env | grep -v '//' | grep -v '#'
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- Pool instances are initialized once at application startup and reused across request handlers
- Graceful shutdown logic calls pool.end() to drain connections before process termination

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All R-PGPOOL rules are mandatory for database access code and MUST be verified before accepting any changes to PostgreSQL connection handling.
</enforcement>