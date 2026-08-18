# Define Express HTTP Service Boundaries with CORS and Route Handlers: Database Connection Strings

These rules are ALWAYS ACTIVE for Express-based HTTP services with route handlers, services requiring CORS configuration for browser-based clients, applications using PostgreSQL connection pooling with environment-based configuration, and endpoints that coordinate database operations with HTTP responses.

### Rules

- **R-CORS-001** MUST: Database connection strings and sensitive configuration MUST be sourced from process.env variables, never hardcoded.
- **R-CORS-002** MUST: Register CORS middleware before route handlers using app.use(cors({allowedHeaders: [...]})) to ensure proper request filtering.
- **R-CORS-003** MUST: Use parameterized queries (pool.query({text: '...', values: [...]})) for all database operations to prevent SQL injection.
- **R-CORS-004** MUST: Configure PostgreSQL connection pool with SSL settings: {connectionString: process.env.DATABASE_URL, ssl: {rejectUnauthorized: false}}.
- **R-CORS-005** SHOULD: Structure route handlers with async/await and send JSON responses using res.send({...}) for consistent API contracts.
- **R-CORS-006** SHOULD: Validate environment variables at application startup before initializing database connections or starting HTTP server.

### Verify

```bash
# Verify CORS middleware configuration
grep -r 'app.use(cors' server/src/ && grep -r 'allowedHeaders' server/src/

# Verify environment variable usage for database connection
grep -r 'process.env.DATABASE_URL' server/src/ && grep -r 'ssl:' server/src/

# Verify route handler definitions
grep -r 'app.get\|app.post\|app.put\|app.delete' server/src/ | grep -v node_modules

# Verify no hardcoded credentials
grep -r 'postgresql://\|mongodb://\|mysql://' server/src/ | grep -v process.env | grep -v '.env.example'
```

**Accept when:**
- CORS middleware is configured with explicit allowedHeaders array before route definitions
- All database connection strings are sourced from process.env variables with SSL configuration
- Route handlers use Express app.get/post/put/delete methods with explicit path strings and async handlers
- No hardcoded credentials or connection strings are present in source code
- Parameterized queries are used for all database operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline validation.
</enforcement>