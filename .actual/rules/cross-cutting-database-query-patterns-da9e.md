# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Database Query Patterns

These rules are ALWAYS ACTIVE for all Node.js/TypeScript files that initialize PostgreSQL connections, execute database queries, or manage connection pooling in the primary datastore.

### Rules

- **R-PGPOOL-001** MUST: All PostgreSQL Pool instantiations include an ssl configuration object with an explicit rejectUnauthorized setting.
- **R-PGPOOL-002** MUST: All database queries with dynamic values use parameterized queries with text and values properties; no string concatenation of user-supplied data into query strings.
- **R-PGPOOL-003** MUST: DATABASE_URL is sourced from process.env and never hardcoded in source files.
- **R-PGPOOL-004** SHOULD: Database query patterns (SELECT, INSERT, UPDATE) SHOULD use the query method with text and values properties for clarity and type safety.
- **R-PGPOOL-005** SHOULD: Initialize the Pool instance once at application startup and reuse it across all request handlers to avoid creating multiple pools.
- **R-PGPOOL-006** SHOULD: Implement graceful shutdown by calling pool.end() to drain connections before process termination.

### Verify

```bash
# Verify all Pool instantiations include ssl configuration with explicit rejectUnauthorized setting
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Count parameterized queries using text and values properties
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from process.env
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Detect any hardcoded connection strings or credentials
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v process.env | grep -v '//' | grep -v '#'
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- No hardcoded connection strings or credentials are present in source code

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PGPOOL-001, R-PGPOOL-002, and R-PGPOOL-003 rules are mandatory and must be verified before accepting code changes. Violations must be flagged for remediation.
</enforcement>