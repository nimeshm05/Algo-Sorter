# Source Runtime Configuration from Environment Variables for Database and Service Binding: Sensitive Configuration Values

These rules are ALWAYS ACTIVE for all TypeScript and JavaScript source files in the server application, particularly those handling database connections, service initialization, and configuration management.

### Rules

- **R-CONFIG-001** MUST NOT: Sensitive configuration values including database credentials, connection strings, and API keys MUST NOT be committed to version control in plaintext.
- **R-CONFIG-002** MUST: All database connection strings and service ports MUST be sourced from process.env variables at application initialization.
- **R-CONFIG-003** MUST: Application startup code MUST validate presence of required environment variables before initializing database pools or HTTP servers.
- **R-CONFIG-004** SHOULD: Use a configuration validation library to enforce required variables and type constraints at startup rather than discovering issues at runtime.
- **R-CONFIG-005** SHOULD: Document all required environment variables in README or deployment documentation and provide .env.example with placeholder values for local development.

### Verify

```bash
# Verify environment variable usage for database and port configuration
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Confirm database connection uses env vars
grep -r 'connectionString.*process\.env' server/src/

# Ensure no hardcoded connection strings in source files
! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js'

# Scan for common credential patterns
git-secrets --scan || truffleHog filesystem . --json
```

**Accept when:**
- All database connection strings and service ports are sourced from process.env variables in server/src/index.ts
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers
- No credential patterns are detected by static analysis tools in the codebase
- Configuration validation occurs at application initialization with clear error messages for missing variables

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for pull request acceptance. Violations block merge until remediated. Security team notification is triggered for any credential exposure incidents.
</enforcement>