# Define Express HTTP Service Boundaries with CORS and Route Handlers: Middleware Registration Occur

These rules are ALWAYS ACTIVE for Express-based HTTP services with route handlers, services requiring CORS configuration for browser-based clients, applications using PostgreSQL connection pooling with environment-based configuration, and endpoints that coordinate database operations with HTTP responses.

### Rules

- **R-EX-001** SHOULD: Middleware registration SHOULD occur before route definitions to ensure proper request processing order.
- **R-EX-002** MUST: Register CORS middleware before route handlers using `app.use(cors({allowedHeaders: [...]}))` to ensure proper request filtering.
- **R-EX-003** MUST: Use parameterized queries (`pool.query({text: '...', values: [...]})`) for all database operations to prevent SQL injection.
- **R-EX-004** MUST: Configure PostgreSQL connection pool with SSL settings: `{connectionString: process.env.DATABASE_URL, ssl: {rejectUnauthorized: false}}`.
- **R-EX-005** MUST: Structure route handlers with async/await and send JSON responses using `res.send({...})` for consistent API contracts.
- **R-EX-006** MUST: Validate environment variables at application startup before initializing database connections or starting HTTP server.
- **R-EX-007** MUST: All database connection strings MUST be sourced from `process.env` variables with SSL configuration.
- **R-EX-008** MUST: CORS middleware MUST be configured with explicit `allowedHeaders` array before route definitions.

### Verify

```bash
# Verify CORS middleware is configured with allowedHeaders before route definitions
grep -r 'app.use(cors' server/src/ && grep -r 'allowedHeaders' server/src/

# Verify environment variables are used for database configuration
grep -r 'process.env.DATABASE_URL' server/src/ && grep -r 'ssl:' server/src/

# Verify route handlers use Express methods with explicit paths
grep -r 'app.get\|app.post\|app.put\|app.delete' server/src/ | grep -v node_modules

# Verify no hardcoded credentials or connection strings
grep -r 'postgresql://\|mongodb://\|mysql://' server/src/ | grep -v process.env | grep -v '.env.example'
```

**Accept when:**
- CORS middleware is configured with explicit `allowedHeaders` array before route definitions
- All database connection strings are sourced from `process.env` variables with SSL configuration
- Route handlers use Express `app.get/post/put/delete` methods with explicit path strings and async handlers
- Parameterized queries are used for all database operations
- Environment variables are validated at application startup
- No hardcoded credentials or connection strings are present in source code

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for Express HTTP services within scope. Code review MUST block merge if CORS configuration is missing or misconfigured. CI pipeline MUST fail if hardcoded credentials or connection strings are detected. Security team MUST be notified for violations involving SSL or secrets handling.
</enforcement>