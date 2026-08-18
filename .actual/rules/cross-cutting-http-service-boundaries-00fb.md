# Define Express HTTP Service Boundaries with CORS and Route Handlers: Http Service Boundaries

These rules are ALWAYS ACTIVE for Express-based HTTP services with route handlers, services requiring CORS configuration for browser-based clients, applications using PostgreSQL connection pooling with environment-based configuration, and endpoints that coordinate database operations with HTTP responses.

### Rules

- **R-EX-001** MUST: HTTP service boundaries MUST be defined using Express route handlers with explicit path definitions (e.g., app.get('/', handler)).
- **R-EX-002** MUST: CORS middleware MUST be registered before route handlers using app.use(cors({allowedHeaders: [...]})) to ensure proper request filtering.
- **R-EX-003** MUST: All database connection strings MUST be sourced from process.env variables with SSL configuration.
- **R-EX-004** MUST: All database operations MUST use parameterized queries (pool.query({text: '...', values: [...]})) to prevent SQL injection.
- **R-EX-005** MUST: Route handlers MUST use async/await and send JSON responses using res.send({...}) for consistent API contracts.
- **R-EX-006** MUST: Environment variables MUST be validated at application startup before initializing database connections or starting HTTP server.
- **R-EX-007** SHOULD: CORS allowedHeaders configuration SHOULD be maintained with explicit origin restrictions rather than wildcard patterns in production environments.
- **R-EX-008** SHOULD: SSL certificate validation SHOULD use proper CA certificates or connection string SSL parameters rather than rejectUnauthorized: false.

### Verify

```bash
# Verify CORS middleware configuration
grep -r 'app.use(cors' server/src/ && grep -r 'allowedHeaders' server/src/

# Verify environment variable usage for database configuration
grep -r 'process.env.DATABASE_URL' server/src/ && grep -r 'ssl:' server/src/

# Verify Express route handler definitions
grep -r 'app.get\|app.post\|app.put\|app.delete' server/src/ | grep -v node_modules

# Verify no hardcoded credentials or connection strings
grep -r 'postgresql://\|mongodb://\|mysql://' server/src/ | grep -v process.env | grep -v '.env.example'
```

**Accept when:**
- CORS middleware is configured with explicit allowedHeaders array before route definitions
- All database connection strings are sourced from process.env variables with SSL configuration
- Route handlers use Express app.get/post/put/delete methods with explicit path strings and async handlers
- All database operations use parameterized queries with values arrays
- No hardcoded credentials or connection strings are present in source code
- Environment variable validation occurs at application startup

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation.
</enforcement>