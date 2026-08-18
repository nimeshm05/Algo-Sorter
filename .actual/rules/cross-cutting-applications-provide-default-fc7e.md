# Source Runtime Configuration from Environment Variables for Database and Service Binding: Applications Provide Default

These rules are ALWAYS ACTIVE for all application code that requires runtime configuration for database connections, service bindings, and environment-specific parameters.

### Rules

- **R-ENV-001** MAY: Applications MAY provide default values for non-sensitive configuration when environment variables are absent.
- **R-ENV-002** MUST: All database connection strings and credentials MUST be sourced from environment variables, never hardcoded in source code.
- **R-ENV-003** MUST: All service port bindings and host addresses MUST be sourced from environment variables at application initialization.
- **R-ENV-004** MUST: Application startup code MUST validate the presence and format of required environment variables before initializing database pools or HTTP servers.
- **R-ENV-005** SHOULD: Applications SHOULD use a configuration validation library to enforce required variables and type constraints at startup.
- **R-ENV-006** SHOULD: All required environment variables SHOULD be documented in README or deployment documentation with .env.example templates provided.

### Verify

```bash
# Verify environment variable usage for database and port configuration
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Confirm database connection uses environment variables
grep -r 'connectionString.*process\.env' server/src/

# Ensure no hardcoded connection strings in source files
! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js'

# Verify no hardcoded credentials patterns
git-secrets --scan
truffleHog filesystem . --json
```

**Accept when:**
- All database connection strings and service ports are sourced from process.env variables in application initialization code
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers
- Static analysis tools (git-secrets, truffleHog) detect no credential patterns in source code
- Configuration management follows twelve-factor application principles

<enforcement>
Claude Code MUST NOT skip or defer verification. All database and service binding configuration MUST be externalized to environment variables. Hardcoded credentials or connection strings are a critical security violation and MUST trigger CI pipeline failure and security team notification.
</enforcement>