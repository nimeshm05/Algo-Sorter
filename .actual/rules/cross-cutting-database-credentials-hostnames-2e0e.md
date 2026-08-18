# Use Environment Variables for Runtime Database Configuration: Database Credentials Hostnames

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in the application, particularly those handling database connections, server initialization, and configuration management.

### Rules

- **R-ENVDB-001** MUST_NOT: Database credentials, hostnames, or connection parameters MUST_NOT be hardcoded in application source code.
- **R-ENVDB-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-ENVDB-003** MUST: Server port configuration MUST be sourced from `process.env.PORT`.
- **R-ENVDB-004** MUST: Startup validation MUST check for required environment variables and provide clear error messages indicating which variables are missing.
- **R-ENVDB-005** SHOULD: Implement logging filters to redact environment variable values and avoid echoing configuration in error responses.
- **R-ENVDB-006** SHOULD: Document all required environment variables in README.md with example values using placeholder credentials, not real ones.
- **R-ENVDB-007** SHOULD: Use a configuration validation library like joi or zod to validate environment variables at startup with clear error messages.

### Verify

```bash
# Check for DATABASE_URL environment variable usage
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Check for PORT environment variable usage
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded PostgreSQL connection strings with credentials
grep -rE '(postgresql://|postgres://).*@.*:.*/' server/src/ --include='*.ts' --include='*.js' | grep -v process.env || echo 'No hardcoded connection strings found'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files
- Server port configuration is sourced from `process.env.PORT`
- No grep matches for hardcoded PostgreSQL connection strings containing credentials in TypeScript or JavaScript source files
- Environment variable validation is implemented at application startup
- Required environment variables are documented with placeholder examples

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must execute successfully before accepting changes to database configuration or server initialization code.
</enforcement>