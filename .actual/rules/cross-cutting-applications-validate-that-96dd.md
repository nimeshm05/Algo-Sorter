# Use Environment Variables for Runtime Database Configuration: Applications Validate That

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in the application that handle database connections or server configuration.

### Rules

- **R-ENV-001** SHOULD: Applications SHOULD validate that required environment variables are present at startup before attempting database connections.
- **R-ENV-002** MUST: All database connection strings MUST be sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files.
- **R-ENV-003** MUST: Server port configuration MUST be sourced from `process.env.PORT`.
- **R-ENV-004** MUST: No hardcoded PostgreSQL connection strings containing credentials shall appear in TypeScript or JavaScript source files.
- **R-ENV-005** SHOULD: Applications SHOULD use a configuration validation library (joi, zod) to validate environment variables at startup with clear error messages.
- **R-ENV-006** SHOULD: Applications SHOULD document all required environment variables in README.md with example values using placeholder credentials.

### Verify

```bash
# Check for DATABASE_URL sourced from process.env
grep -r 'process\.env\.DATABASE_URL' server/src/ | grep -v 'node_modules'

# Check for PORT sourced from process.env
grep -r 'process\.env\.PORT' server/src/ | grep -v 'node_modules'

# Verify no hardcoded PostgreSQL connection strings with credentials
grep -rE '(postgresql://|postgres://).*@.*:.*/' server/src/ --include='*.ts' --include='*.js' | grep -v process.env || echo 'No hardcoded connection strings found'
```

**Accept when:**
- All database connection strings are sourced from `process.env.DATABASE_URL` with no hardcoded credentials in source files
- Server port configuration is sourced from `process.env.PORT`
- No grep matches for hardcoded PostgreSQL connection strings containing credentials in TypeScript or JavaScript source files
- Environment variable validation is implemented at application startup with clear error messaging
- Required environment variables are documented in project README with placeholder examples

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory checks during code review and must pass before merge approval.
</enforcement>