# Adopt Express Middleware Stack with CORS and JSON Parsing for HTTP Request Processing: Applications Use Express

These rules are ALWAYS ACTIVE for all Node.js/Express application files that configure HTTP request processing middleware, particularly `server/src/index.ts` and equivalent entry points.

### Rules

- **R-EX-001** MUST: Applications MUST use express.json() middleware via app.use(express.json()) to parse JSON request bodies before route handlers execute.
- **R-EX-002** MUST: CORS middleware MUST be configured using app.use(cors()) before any route definitions (app.get, app.post, etc.) to ensure preprocessing occurs for all requests.
- **R-EX-003** MUST: All middleware registrations MUST appear before the first route handler definition in the source file.
- **R-EX-004** SHOULD: CORS configuration SHOULD use the 'origin' option rather than 'allowedHeaders' to properly restrict cross-origin access: cors({ origin: ['https://suraj-gov.github.io/sorter', 'http://localhost:3000'] }).
- **R-EX-005** SHOULD: Document the middleware execution order and the purpose of each middleware component for future maintainers.

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
- CORS middleware is configured using app.use(cors()) before route definitions
- JSON parsing middleware is configured using app.use(express.json()) before route definitions
- All middleware registrations appear before the first route handler definition in the source file
- CORS configuration uses the 'origin' option to restrict allowed origins

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-EX rules must be validated before approving Express application middleware configuration.
</enforcement>