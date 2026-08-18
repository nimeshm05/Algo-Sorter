# Define Express HTTP Service Boundaries with CORS and Route Handlers: Services Expose Health

These rules are ALWAYS ACTIVE for all Express-based HTTP services with route handlers, CORS configuration, and PostgreSQL connection pooling using environment-based configuration.

### Rules

- **R-EX-001** MAY: Services MAY expose health check or status endpoints that aggregate database query results.

### Verify

```bash
# Verify CORS middleware configuration with allowedHeaders
grep -r 'app.use(cors' server/src/ && grep -r 'allowedHeaders' server/src/

# Verify environment variable usage for database configuration
grep -r 'process.env.DATABASE_URL' server/src/ && grep -r 'ssl:' server/src/

# Verify Express route handler definitions
grep -r 'app.get\|app.post\|app.put\|app.delete' server/src/ | grep -v node_modules
```

**Accept when:**
- CORS middleware is configured with explicit allowedHeaders array before route definitions
- All database connection strings are sourced from process.env variables with SSL configuration
- Route handlers use Express app.get/post/put/delete methods with explicit path strings and async handlers
- Health check endpoints aggregate database query results when exposed

<enforcement>
Claude Code MUST NOT skip or defer verification. All CORS configuration, environment variable usage, and route handler patterns MUST be validated before accepting changes to Express HTTP services.
</enforcement>