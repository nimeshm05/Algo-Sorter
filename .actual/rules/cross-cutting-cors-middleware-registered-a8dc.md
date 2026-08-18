# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Cors Middleware Registered

These rules are ALWAYS ACTIVE for all Express.js middleware configuration files and HTTP endpoint implementations that handle cross-origin requests in the visitor tracking API.

### Rules

- **R-CORS-001** MUST: The CORS middleware MUST be registered in the Express.js middleware stack via app.use() before route handlers to ensure policy enforcement on all endpoints.
- **R-CORS-002** MUST: CORS configuration MUST use an explicit origin allowlist (not wildcard) to control which origins can access the API.
- **R-CORS-003** MUST: Production CORS configuration MUST exclude development origins (localhost, 127.0.0.1) and only permit HTTPS origins.
- **R-CORS-004** SHOULD: CORS configuration SHOULD be externalized to environment variables (e.g., CORS_ALLOWED_ORIGINS) rather than hardcoded in application code.
- **R-CORS-005** SHOULD: CORS middleware configuration SHOULD use the 'origin' parameter for origin validation rather than 'allowedHeaders'.

### Verify

```bash
# Check that cors middleware is configured with explicit origin allowlist (not wildcard)
grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"

# Verify CORS headers are correctly set for allowed origins
curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"

# Check for localhost in CORS config (should not appear in production)
grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"

# Verify cors middleware is registered before route handlers
grep -n "app.use(cors" server/src/index.ts | head -1
grep -n "app.get\|app.post\|app.put\|app.delete" server/src/index.ts | head -1
```

**Accept when:**
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins
- Requests from unauthorized origins are blocked by CORS policy
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol
- CORS configuration uses the 'origin' parameter for origin validation
- Integration tests verify CORS behavior for both allowed and blocked origins

<enforcement>
Claude Code MUST NOT skip or defer verification of CORS middleware configuration. All R-CORS rules marked MUST are mandatory and must be verified before accepting changes to CORS configuration or middleware registration.
</enforcement>