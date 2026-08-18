# Use Environment Variables for Runtime Database Configuration: Ssl Configuration Database

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in the server application that handle database connectivity and runtime configuration.

### Rules

- **R-ENVDB-001** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-ENVDB-002** MUST: Server port configuration MUST be sourced from `process.env.PORT`.
- **R-ENVDB-003** SHOULD: SSL configuration for database connections SHOULD be included in the connection pool initialization with `rejectUnauthorized` set to false for cloud-hosted databases.
- **R-ENVDB-004** MUST: No hardcoded PostgreSQL connection strings containing credentials (matching pattern `postgresql://` or `postgres://` with user:password@host:port) MUST appear in TypeScript or JavaScript source files.

### Verify

```bash
# Check for DATABASE_URL environment variable usage
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Check for PORT environment variable usage
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded connection strings with credentials exist
grep -rE '(postgresql://|postgres://).*@.*:.*/' server/src/ --include='*.ts' --include='*.js' | grep -v process.env || echo 'No hardcoded connection strings found'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files
- Server port configuration is sourced from `process.env.PORT`
- No grep matches for hardcoded PostgreSQL connection strings containing credentials in TypeScript or JavaScript source files
- SSL configuration is present in the connection pool initialization when connecting to cloud-hosted databases

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands MUST execute successfully before accepting changes to database configuration or connection handling code.
</enforcement>