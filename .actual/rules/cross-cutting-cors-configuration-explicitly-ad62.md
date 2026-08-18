# Adopt Express Middleware Stack with CORS and JSON Parsing for HTTP Request Processing: Cors Configuration Explicitly

These rules are ALWAYS ACTIVE for all Express.js HTTP server configuration files that register middleware and route handlers.

### Rules

- **R-CORS-001** SHOULD: CORS configuration SHOULD explicitly enumerate allowed origins rather than using wildcard (*) to maintain security boundaries.
- **R-CORS-002** MUST: Register middleware using app.use() before any route definitions (app.get, app.post, etc.) to ensure preprocessing occurs for all requests.
- **R-CORS-003** MUST: Verify CORS configuration uses the 'origin' option rather than 'allowedHeaders' to properly restrict cross-origin access.
- **R-CORS-004** MUST: JSON parsing middleware MUST be configured using app.use(express.json()) before route definitions.

### Verify

```bash
# Check for CORS middleware registration
grep -n 'app\.use(cors(' server/src/index.ts

# Check for JSON parsing middleware registration
grep -n 'app\.use(express\.json())' server/src/index.ts

# Verify middleware appears before route definitions
grep -B5 'app\.get\|app\.post' server/src/index.ts | grep -c 'app\.use'
```

**Accept when:**
- CORS middleware is configured using app.use(cors()) before route definitions
- JSON parsing middleware is configured using app.use(express.json()) before route definitions
- All middleware registrations appear before the first route handler definition in the source file
- CORS configuration explicitly specifies allowed origins: cors({ origin: ['https://suraj-gov.github.io/sorter', 'http://localhost:3000'] })

<enforcement>
Claude Code MUST NOT skip or defer verification. All middleware MUST be registered before route definitions, and CORS configuration MUST use explicit origin enumeration rather than wildcards or allowedHeaders-only patterns.
</enforcement>