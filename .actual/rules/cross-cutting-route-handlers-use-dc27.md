# Define Express HTTP Service Boundaries with CORS and Route Handlers: Route Handlers Use

These rules are ALWAYS ACTIVE for all Express-based HTTP services with route handlers, CORS configuration, and PostgreSQL connection pooling using environment-based configuration.

### Rules

- **R-EX-001** SHOULD: Route handlers SHOULD use async/await patterns for database operations and send structured JSON responses.
- **R-EX-002** MUST: Register CORS middleware before route handlers using `app.use(cors({allowedHeaders: [...]}))` to ensure proper request filtering.
- **R-EX-003** MUST: Use parameterized queries (`pool.query({text: '...', values: [...]})`) for all database operations to prevent SQL injection.
- **R-EX-004** MUST: Configure PostgreSQL connection pool with SSL settings: `{connectionString: process.env.DATABASE_URL, ssl: {rejectUnauthorized: false}}`.
- **R-EX-005** MUST: Structure route handlers with async/await and send JSON responses using `res.send({...})` for consistent API contracts.
- **R-EX-006** MUST: Validate environment variables at application startup before initializing database connections or starting HTTP server.

### Verify

```bash
# Verify CORS middleware configuration
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
- All database queries use parameterized query syntax with separate text and values properties
- Environment variables are validated at application startup with clear error messages

<enforcement>
Clause Code MUST NOT skip or defer verification. Static analysis scanning for hardcoded credentials or connection strings is mandatory. Integration tests validating CORS headers for approved and blocked origins are required. Security scanning tools checking SSL configuration and secrets handling must pass before merge.
</enforcement>