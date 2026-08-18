# Adopt Express Middleware Stack with CORS and JSON Parsing for HTTP Request Processing: Middleware Configuration Applied

These rules are ALWAYS ACTIVE for all Express application server files that configure HTTP middleware and route handlers.

### Rules

- **R-MW-001** MUST: Middleware configuration MUST be applied using app.use() before route definitions to ensure preprocessing occurs for all endpoints.

### Verify

```bash
# Check for CORS middleware configuration
grep -n 'app\.use(cors(' server/src/index.ts

# Check for JSON parsing middleware configuration
grep -n 'app\.use(express\.json())' server/src/index.ts

# Verify middleware registrations precede route definitions
grep -B5 'app\.get\|app\.post' server/src/index.ts | grep -c 'app\.use'
```

**Accept when:**
- CORS middleware is configured using app.use(cors()) before route definitions
- JSON parsing middleware is configured using app.use(express.json()) before route definitions
- All middleware registrations appear before the first route handler definition in the source file

<enforcement>
Claude Code MUST NOT skip or defer verification. Middleware configuration order is critical to security and request processing; violations must be caught before merge.
</enforcement>