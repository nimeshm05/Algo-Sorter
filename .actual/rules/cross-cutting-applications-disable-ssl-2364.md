# Configure PostgreSQL Connection Pooling with SSL for Primary Datastore: Applications Disable Ssl

These rules are ALWAYS ACTIVE for all Node.js/TypeScript applications using PostgreSQL as the primary datastore through the 'pg' library with connection pooling.

### Rules

- **R-PGSSL-001** MAY: Applications MAY disable SSL certificate validation (rejectUnauthorized: false) only when connecting to managed database services that use self-signed certificates.
- **R-PGSSL-002** MUST: All PostgreSQL Pool instantiations include an ssl configuration object with explicit rejectUnauthorized setting.
- **R-PGSSL-003** MUST: All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation.
- **R-PGSSL-004** MUST: DATABASE_URL is sourced from process.env and not hardcoded in any source files.
- **R-PGSSL-005** MUST: Initialize the Pool instance once at application startup and reuse it across all request handlers to avoid creating multiple pools.
- **R-PGSSL-006** MUST: Implement graceful shutdown by calling pool.end() to drain connections before process termination.
- **R-PGSSL-007** SHOULD: Configure pool size using environment variables (e.g., PGMAXCONNECTIONS) to tune for specific deployment environments and database limits.
- **R-PGSSL-008** SHOULD: For managed database services requiring rejectUnauthorized: false, add inline comments explaining the hosting provider's certificate configuration.

### Verify

```bash
# Verify all PostgreSQL Pool instantiations include ssl configuration
grep -r 'new Pool' --include='*.ts' --include='*.js' | grep -E 'ssl.*rejectUnauthorized'

# Count parameterized queries using text/values pattern
grep -r 'pool\.query.*\$[0-9]' --include='*.ts' --include='*.js' | wc -l

# Verify DATABASE_URL is sourced from process.env
grep -r 'process\.env\.DATABASE_URL' --include='*.ts' --include='*.js'

# Verify no hardcoded connection strings
grep -r 'postgresql://' --include='*.ts' --include='*.js' | grep -v 'process.env'
```

**Accept when:**
- All PostgreSQL Pool instantiations include ssl configuration object with explicit rejectUnauthorized setting
- All database queries with dynamic values use parameterized queries (text and values properties) with no string concatenation
- DATABASE_URL is sourced from process.env and not hardcoded in any source files
- Pool is initialized once at application startup and reused across all request handlers
- Graceful shutdown calls pool.end() before process termination
- For managed database services requiring rejectUnauthorized: false, inline comments document the hosting provider's certificate configuration

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for PostgreSQL connection implementations unless an approved exception (EXC-001 or EXC-002) is documented in code comments.
</enforcement>