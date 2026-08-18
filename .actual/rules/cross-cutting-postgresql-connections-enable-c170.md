# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Postgresql Connections Enable

These rules are ALWAYS ACTIVE for all Node.js/TypeScript applications using PostgreSQL as the primary datastore, including connection initialization, query execution, and environment-based configuration patterns.

### Rules

- **R-PGPOOL-001** MUST: PostgreSQL connections MUST enable SSL/TLS encryption by configuring the ssl property in Pool options.
- **R-PGPOOL-002** MUST: All PostgreSQL Pool instantiations MUST include an explicit ssl configuration object with rejectUnauthorized setting documented.
- **R-PGPOOL-003** MUST: All database queries with dynamic values MUST use parameterized queries (text and values properties) with no string concatenation.
- **R-PGPOOL-004** MUST: DATABASE_URL MUST be sourced from process.env and MUST NOT be hardcoded in any source files.
- **R-PGPOOL-005** MUST: Pool instances MUST be initialized once at application startup and reused across all request handlers.
- **R-PGPOOL-006** SHOULD: Pool size SHOULD be configured using environment variables (e.g., PGMAXCONNECTIONS) to tune for specific deployment environments.
- **R-PGPOOL-007** SHOULD: Applications SHOULD implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-PGPOOL-008** SHOULD: For managed database services requiring rejectUnauthorized: false, inline comments SHOULD explain the hosting provider's certificate configuration.

### Verify

```bash
# Verify all PostgreSQL Pool instantiations include ssl configuration
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Count parameterized queries using text/values pattern
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from environment, not hardcoded
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Verify no hardcoded connection strings
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v 'process.env' | grep -v '//' || echo 'No hardcoded connection strings found'
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- Pool instances are initialized once at application startup and reused across request handlers
- Graceful shutdown is implemented with pool.end() called before process termination
- For managed services with rejectUnauthorized: false, inline comments document the hosting provider requirement

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-PGPOOL rules marked MUST are mandatory and must be verified before code acceptance. Violations detected by static analysis or code review MUST block merge and trigger security team notification.
</enforcement>