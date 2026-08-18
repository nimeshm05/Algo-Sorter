# Use Environment Variables for Runtime Database Configuration: Server Port Configuration

These rules are ALWAYS ACTIVE for all server application code that configures runtime database connections and server port binding, particularly in TypeScript/JavaScript files under `server/src/`.

### Rules

- **R-ENV-001** MUST: Server port configuration MUST be sourced from the PORT environment variable using process.env.PORT
- **R-ENV-002** MUST: Database connection strings MUST be sourced from the DATABASE_URL environment variable using process.env.DATABASE_URL with no hardcoded credentials in source files
- **R-ENV-003** SHOULD: Implement startup validation that checks for required environment variables and provides clear error messages indicating which variables are missing
- **R-ENV-004** SHOULD: Use a .env file with dotenv library for local development to avoid manually setting environment variables in each shell session
- **R-ENV-005** SHOULD: Document all required environment variables in README.md with example values using placeholder credentials, not real ones
- **R-ENV-006** SHOULD: Consider using a configuration validation library like joi or zod to validate environment variables at startup with clear error messages

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
- All database connection strings are sourced from process.env.DATABASE_URL with no hardcoded credentials in source files
- Server port configuration is sourced from process.env.PORT
- No grep matches for hardcoded PostgreSQL connection strings containing credentials in TypeScript or JavaScript source files
- Environment variable usage is consistent across all server startup code

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to server port or database configuration code.
</enforcement>