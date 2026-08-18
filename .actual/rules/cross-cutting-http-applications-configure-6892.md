# Adopt Express Middleware Stack with CORS and JSON Parsing for HTTP Request Processing: Http Applications Configure

These rules are ALWAYS ACTIVE for all HTTP application files that configure Express middleware stacks, particularly server entry points and application initialization files.

### Rules

- **R-HTTP-001** MUST: All HTTP applications MUST configure CORS middleware using `app.use(cors())` with explicit origin configuration before any route definitions.
- **R-HTTP-002** MUST: JSON parsing middleware MUST be configured using `app.use(express.json())` before any route definitions.
- **R-HTTP-003** MUST: All middleware registrations using `app.use()` MUST appear before the first route handler definition (`app.get`, `app.post`, etc.).
- **R-HTTP-004** SHOULD: CORS configuration SHOULD use the `origin` option to specify allowed origins (e.g., `cors({ origin: ['https://suraj-gov.github.io/sorter', 'http://localhost:3000'] })`) rather than `allowedHeaders`.
- **R-HTTP-005** SHOULD: Middleware execution order SHOULD be documented with inline comments explaining the purpose of each middleware component.

### Verify

```bash
# Check for CORS middleware configuration
grep -n 'app\.use(cors(' server/src/index.ts

# Check for JSON parsing middleware configuration
grep -n 'app\.use(express\.json())' server/src/index.ts

# Verify middleware appears before route definitions
grep -B5 'app\.get\|app\.post' server/src/index.ts | grep -c 'app\.use'
```

**Accept when:**
- CORS middleware is configured using `app.use(cors())` before any route definitions
- JSON parsing middleware is configured using `app.use(express.json())` before any route definitions
- All middleware registrations appear before the first route handler definition in the source file
- CORS configuration uses the `origin` option to specify allowed origins

<enforcement>
Claude Code MUST NOT skip or defer verification of middleware configuration order and presence. Violations must be flagged during code review and CI pipeline checks.
</enforcement>