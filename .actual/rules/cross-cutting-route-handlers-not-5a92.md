# Adopt Express Middleware Stack with CORS and JSON Parsing for HTTP Request Processing: Route Handlers Not

These rules are ALWAYS ACTIVE for all Express.js HTTP server files that define middleware configuration and route handlers, particularly `server/src/index.ts` and similar entry points.

### Rules

- **R-MIDDLEWARE-001** MUST_NOT: Route handlers MUST_NOT implement their own CORS headers or JSON parsing logic when middleware is configured.

### Verify

```bash
# Verify CORS middleware is configured
grep -n 'app\.use(cors(' server/src/index.ts

# Verify JSON parsing middleware is configured
grep -n 'app\.use(express\.json())' server/src/index.ts

# Verify middleware appears before route definitions
grep -B5 'app\.get\|app\.post' server/src/index.ts | grep -c 'app\.use'
```

**Accept when:**
- CORS middleware is configured using `app.use(cors())` before route definitions
- JSON parsing middleware is configured using `app.use(express.json())` before route definitions
- All middleware registrations appear before the first route handler definition in the source file
- Route handlers do not contain custom CORS header logic (e.g., `res.setHeader('Access-Control-Allow-Origin', ...)`)
- Route handlers do not contain custom JSON parsing logic (e.g., manual `JSON.parse()` calls on request bodies)

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must execute successfully and all accept criteria must be satisfied before approving changes to Express middleware configuration or route handler implementations.
</enforcement>