# Define Express HTTP Service Boundaries with CORS and Route Handlers: Cors Middleware Configured

These rules are ALWAYS ACTIVE for Express-based HTTP services with route handlers, services requiring CORS configuration for browser-based clients, and applications using PostgreSQL connection pooling with environment-based configuration.

### Rules

- **R-CORS-001** MUST: CORS middleware MUST be configured with explicit allowedHeaders to restrict cross-origin requests to approved origins only.

### Verify

```bash
# Verify CORS middleware is configured with allowedHeaders
grep -r 'app.use(cors' server/src/ && grep -r 'allowedHeaders' server/src/

# Verify environment variables are used for database configuration
grep -r 'process.env.DATABASE_URL' server/src/ && grep -r 'ssl:' server/src/

# Verify route handlers use Express methods with explicit paths
grep -r 'app.get\|app.post\|app.put\|app.delete' server/src/ | grep -v node_modules
```

**Accept when:**
- CORS middleware is configured with explicit allowedHeaders array before route definitions
- All database connection strings are sourced from process.env variables with SSL configuration
- Route handlers use Express app.get/post/put/delete methods with explicit path strings and async handlers
- No hardcoded credentials or connection strings are present in the codebase

<enforcement>
Clause Code MUST NOT skip or defer verification of CORS middleware configuration, environment variable usage, and route handler patterns. All three verification commands must pass before accepting changes to Express HTTP service boundaries.
</enforcement>