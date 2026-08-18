# Store Database Connection Strings in Environment Variables: Environment Variables Containing

These rules are ALWAYS ACTIVE for all server-side code files that initialize database connections or reference runtime configuration.

### Rules

- **R-ENV-001** MUST: Environment variables containing secrets MUST be documented as required runtime dependencies.
- **R-ENV-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-ENV-003** MUST: PostgreSQL Pool initialization MUST include SSL configuration and reference environment variables.
- **R-ENV-004** MUST: No hardcoded `postgresql://` connection strings outside of environment variable references are permitted in production code.
- **R-ENV-005** SHOULD: A `.env.example` file SHOULD document required environment variables (DATABASE_URL, PORT) without actual credentials.
- **R-ENV-006** SHOULD: Startup validation SHOULD check for DATABASE_URL presence and attempt connection before accepting requests.
- **R-ENV-007** MAY: Test fixtures MAY use hardcoded test database URLs only for isolated unit tests with clearly marked test data.
- **R-ENV-008** MAY: Documentation examples MAY use placeholder values (e.g., `postgresql://user:pass@host:5432/db`) not real credentials.

### Verify

```bash
# Check for process.env.DATABASE_URL usage in server code
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Verify no hardcoded credentials exist outside environment variable references
grep -r 'postgresql://' server/src/ | grep -v 'process.env' | grep -v 'node_modules' && echo 'FAIL: Hardcoded credentials found' || echo 'PASS: No hardcoded credentials'

# Verify PostgreSQL Pool initialization includes environment variable references
grep -r 'new Pool' server/src/ | grep -A 5 'connectionString' | grep 'process.env'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- PostgreSQL Pool initialization includes SSL configuration and references environment variables.
- No grep matches for hardcoded `postgresql://` connection strings outside of environment variable references.
- CI pipeline verification commands pass without detecting hardcoded credentials.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All database connection configuration MUST use environment variables. Hardcoded credentials in production code are a critical security violation and MUST be rejected.
</enforcement>