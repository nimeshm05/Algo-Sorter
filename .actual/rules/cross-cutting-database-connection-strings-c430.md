# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Database Connection Strings

These rules are ALWAYS ACTIVE for all Node.js/TypeScript applications using PostgreSQL as the primary datastore, including connection initialization, query execution, and environment-based configuration patterns.

### Rules

- **R-PGPOOL-001** MUST: Database connection strings MUST be sourced from environment variables (DATABASE_URL) and MUST NOT be hardcoded in source files.
- **R-PGPOOL-002** MUST: All PostgreSQL Pool instantiations MUST include an ssl configuration object with an explicit rejectUnauthorized setting.
- **R-PGPOOL-003** MUST: All database queries with dynamic values MUST use parameterized queries (text and values properties) with no string concatenation.
- **R-PGPOOL-004** MUST: The Pool instance MUST be initialized once at application startup and reused across all request handlers to avoid creating multiple pools.
- **R-PGPOOL-005** MUST: Graceful shutdown MUST be implemented by calling pool.end() to drain connections before process termination.
- **R-PGPOOL-006** SHOULD: Pool size SHOULD be configured using environment variables (e.g., PGMAXCONNECTIONS) to tune for specific deployment environments and database limits.
- **R-PGPOOL-007** SHOULD: Explicit environment variable validation SHOULD be implemented at startup with clear error messages for missing or incorrect DATABASE_URL.
- **R-PGPOOL-008** SHOULD: For managed database services requiring rejectUnauthorized: false, inline comments SHOULD explain the hosting provider's certificate configuration.

### Verify

```bash
# Verify all PostgreSQL Pool instantiations include ssl configuration
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Count parameterized queries using text/values pattern
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from process.env
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Detect hardcoded connection strings or credentials
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v 'process.env' | grep -v '//' | grep -v '/*'
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- No hardcoded connection strings or credentials are present in source code
- Pool instance is initialized once at application startup and reused across request handlers
- Graceful shutdown with pool.end() is implemented before process termination

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PGPOOL rules marked MUST are non-negotiable and MUST be verified before code acceptance. Violations MUST trigger CI pipeline failure and code review blocking.
</enforcement>