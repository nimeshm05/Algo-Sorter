# Use Environment Variables for Runtime Database Configuration: Applications Provide Default

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in the application that handle database connectivity and server configuration.

### Rules

- **R-ENV-001** MUST: Source all database connection strings from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-ENV-002** MUST: Source server port configuration from `process.env.PORT`.
- **R-ENV-003** MAY: Applications MAY provide default values for PORT when the environment variable is not set.
- **R-ENV-004** MUST: Implement startup validation that checks for required environment variables and provides clear error messages indicating which variables are missing.
- **R-ENV-005** MUST: Implement logging filters to redact environment variable values and avoid echoing configuration in error responses.
- **R-ENV-006** SHOULD: Use a .env file with dotenv library for local development to avoid manually setting environment variables in each shell session.
- **R-ENV-007** SHOULD: Document all required environment variables in README.md with example values using placeholder credentials, not real ones.
- **R-ENV-008** SHOULD: Consider using a configuration validation library like joi or zod to validate environment variables at startup with clear error messages.

### Verify

```bash
# Check for DATABASE_URL sourced from environment
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Check for PORT sourced from environment
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded PostgreSQL connection strings with credentials
grep -rE '(postgresql://|postgres://).*@.*:.*/' server/src/ --include='*.ts' --include='*.js' | grep -v process.env || echo 'No hardcoded connection strings found'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files
- Server port configuration is sourced from `process.env.PORT`
- No grep matches for hardcoded PostgreSQL connection strings containing credentials in TypeScript or JavaScript source files
- Startup validation is implemented for required environment variables
- Logging filters are in place to redact sensitive configuration values

<enforcement>
Claude Code MUST NOT skip or defer verification. All database and port configuration MUST be externalized to environment variables. Pull requests containing hardcoded credentials are automatically flagged and blocked from merging.
</enforcement>