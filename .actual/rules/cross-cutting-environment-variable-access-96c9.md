# Source Runtime Configuration from Environment Variables for Database and Service Binding: Environment Variable Access

These rules are ALWAYS ACTIVE for all application initialization code, database connection configuration, and service binding setup in TypeScript and JavaScript source files.

### Rules

- **R-ENV-001** SHOULD: Environment variable access SHOULD be centralized at application initialization to fail fast on missing required configuration.
- **R-ENV-002** MUST: All database connection strings and service ports MUST be sourced from process.env variables, never hardcoded in source code.
- **R-ENV-003** MUST: Application startup code MUST validate presence and format of required environment variables before initializing database pools or HTTP servers.
- **R-ENV-004** SHOULD: Configuration validation SHOULD use a validation library to enforce required variables and type constraints at startup.
- **R-ENV-005** SHOULD: Documentation SHOULD include all required environment variables in README or deployment documentation with .env.example templates.

### Verify

```bash
# Verify environment variable usage for DATABASE_URL and PORT
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Confirm database connection uses env vars
grep -r 'connectionString.*process\.env' server/src/

# Ensure no hardcoded connection strings in source files
! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js'

# Verify no hardcoded credentials patterns
git-secrets --scan
truffleHog filesystem server/src/ --json
```

**Accept when:**
- All database connection strings and service ports are sourced from process.env variables in server/src/index.ts
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers
- Configuration validation occurs at application initialization with clear error messages for missing variables
- Required environment variables are documented with .env.example templates provided

<enforcement>
Claude Code MUST NOT skip or defer verification. All verify commands MUST pass before accepting changes to configuration management code. Static analysis tools MUST be integrated into CI pipeline to detect credential patterns. Pull requests containing configuration violations MUST be blocked from merge.
</enforcement>