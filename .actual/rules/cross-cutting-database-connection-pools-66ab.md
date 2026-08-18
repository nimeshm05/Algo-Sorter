# Source Runtime Configuration from Environment Variables for Database and Service Binding: Database Connection Pools

These rules are ALWAYS ACTIVE for all database connection configuration, service binding setup, and runtime configuration sourcing in the application.

### Rules

- **R-DBPOOL-001** SHOULD: Database connection pools SHOULD be initialized with environment-sourced configuration including SSL settings appropriate to the deployment context.

### Verify

```bash
# Verify environment variable usage for database and port configuration
grep -r 'process\.env\.' server/src/ | grep -E '(DATABASE_URL|PORT)'

# Confirm database connection uses environment variables
grep -r 'connectionString.*process\.env' server/src/

# Ensure no hardcoded connection strings in source code
! grep -r 'postgresql://.*:.*@' server/src/ --include='*.ts' --include='*.js'
```

**Accept when:**
- All database connection strings and service ports are sourced from process.env variables in server/src/index.ts
- No hardcoded credentials or connection strings appear in TypeScript or JavaScript source files
- Application startup code validates presence of required environment variables before initializing database pools or HTTP servers
- PostgreSQL Pool configuration objects use connectionString from environment variables with appropriate SSL settings

<enforcement>
Claude Code MUST NOT skip or defer verification of environment variable sourcing for database connections and service bindings. All database configuration MUST be externalized from source code.
</enforcement>