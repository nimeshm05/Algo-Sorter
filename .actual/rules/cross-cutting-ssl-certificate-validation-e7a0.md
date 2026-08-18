# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Ssl Certificate Validation

These rules are ALWAYS ACTIVE for all Node.js/TypeScript applications using PostgreSQL as the primary datastore, including connection initialization, query execution, and environment-based configuration patterns.

### Rules

- **R-PGSSL-001** SHOULD: SSL certificate validation SHOULD be enabled (rejectUnauthorized: true) unless explicitly documented exceptions exist for development or specific hosting environments.
- **R-PGSSL-002** MUST: All PostgreSQL Pool instantiations MUST include an ssl configuration object with an explicit rejectUnauthorized setting.
- **R-PGSSL-003** MUST: All database queries with dynamic values MUST use parameterized queries (text and values properties) with no string concatenation.
- **R-PGSSL-004** MUST: DATABASE_URL MUST be sourced from process.env and MUST NOT be hardcoded in any source files.
- **R-PGSSL-005** MUST: The Pool instance MUST be initialized once at application startup and reused across all request handlers to avoid creating multiple pools.
- **R-PGSSL-006** MUST: Graceful shutdown MUST be implemented by calling pool.end() to drain connections before process termination.
- **R-PGSSL-007** SHOULD: Pool size SHOULD be configured using environment variables (e.g., PGMAXCONNECTIONS) to tune for specific deployment environments and database limits.
- **R-PGSSL-008** SHOULD: For managed database services requiring rejectUnauthorized: false, inline comments SHOULD explain the hosting provider's certificate configuration and reference the approved exception.

### Verify

```bash
# Verify all PostgreSQL Pool instantiations include ssl configuration
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Verify parameterized query usage (count queries with $N placeholders)
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from environment, not hardcoded
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Verify no hardcoded connection strings
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v 'process.env' | grep -v '//' || echo "No hardcoded connection strings found"
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- Pool instance is initialized once at application startup and reused across request handlers
- Graceful shutdown is implemented with pool.end() called before process termination
- For any rejectUnauthorized: false usage, inline comments document the hosting provider requirement and reference an approved exception

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PGSSL rules MUST be verified before accepting database connection code. Violations in CI pipeline MUST fail the build. Code review MUST block merge if database connections lack proper pooling or SSL settings.
</enforcement>