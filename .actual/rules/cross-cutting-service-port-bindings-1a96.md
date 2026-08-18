# Source Runtime Configuration from Environment Variables for Database and Service Binding: Service Port Bindings

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in `server/src/` that handle database connections, service initialization, and runtime configuration.

### Rules

- **R-ENVCONF-001** MUST: Service port bindings MUST be sourced from environment variables (process.env.PORT) to support dynamic port assignment across deployment environments.
- **R-ENVCONF-002** MUST: All database connection strings MUST be sourced from environment variables (process.env.DATABASE_URL) without hardcoding credentials or connection parameters in source code.
- **R-ENVCONF-003** MUST: Application startup code MUST validate the presence and format of required environment variables before initializing database pools or HTTP servers.
- **R-ENVCONF-004** SHOULD: Use a configuration validation library to enforce required variables and type constraints at startup rather than discovering issues at runtime.
- **R-ENVCONF-005** SHOULD: Document all required environment variables in README or deployment documentation and provide .env.example with placeholder values for local development.

### Verify

```bash
# Verify environment variable usage for DATABASE_URL and PORT
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Confirm database connection uses env vars
grep -r 'connectionString.*process\.env' server/src/

# Ensure no hardcoded connection strings in TypeScript or JavaScript files
! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js'

# Verify no hardcoded credentials patterns
grep -r 'password.*=' server/src/ --include='*.ts' --include='*.js' | grep -v 'process\.env' || true
```

**Accept when:**
- All database connection strings and service ports are sourced from process.env variables in server/src/index.ts
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers
- PostgreSQL Pool configuration objects use connectionString from environment variables, including SSL settings
- No credential patterns (e.g., postgresql://user:password@host) are found in source code

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and must be verified before accepting changes to configuration management or database connection initialization.
</enforcement>