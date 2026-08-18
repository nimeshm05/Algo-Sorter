# Source Runtime Configuration from Environment Variables for Database and Service Binding: Database Connection Strings

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in `server/src/` that handle database connections, service binding, or runtime configuration.

### Rules

- **R-ENVCONFIG-001** MUST: Database connection strings MUST be sourced from environment variables (process.env.DATABASE_URL) rather than hardcoded in application source code.
- **R-ENVCONFIG-002** MUST: Service port bindings MUST be sourced from environment variables (process.env.PORT) rather than hardcoded in application source code.
- **R-ENVCONFIG-003** MUST: Application startup code MUST validate presence of required environment variables before initializing database pools or HTTP servers.
- **R-ENVCONFIG-004** SHOULD: Use a configuration validation library to enforce required variables and type constraints at startup rather than discovering issues at runtime.
- **R-ENVCONFIG-005** SHOULD: Document all required environment variables in README or deployment documentation; provide .env.example with placeholder values for local development.

### Verify

```bash
# Verify environment variable usage for DATABASE_URL and PORT
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Confirm database connection uses env vars
grep -r 'connectionString.*process\.env' server/src/

# Ensure no hardcoded connection strings
! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js'

# Ensure no hardcoded credentials in source
! grep -r 'password.*=.*['\'"\']' server/src/ --include='*.ts' --include='*.js' | grep -v process\.env
```

**Accept when:**
- All database connection strings and service ports are sourced from process.env variables in server/src/index.ts
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers
- PostgreSQL Pool configuration objects use connectionString from environment variables, including SSL settings
- No sensitive values (passwords, API keys, connection strings) are embedded in source code

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and must be checked before accepting changes to database connection or service binding configuration.
</enforcement>