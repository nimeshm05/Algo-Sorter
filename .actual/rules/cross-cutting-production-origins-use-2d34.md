# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Production Origins Use

These rules are ALWAYS ACTIVE for all HTTP endpoints exposed by the Express.js application, cross-origin requests from browser-based clients, and middleware stack initialization in server/src/index.ts.

### Rules

- **R-CORS-001** MUST: Production origins MUST use HTTPS protocol with fully qualified domain names in the allowlist.
- **R-CORS-002** MUST: CORS middleware MUST be configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack.
- **R-CORS-003** MUST: Production configuration MUST exclude development origins (localhost) from the allowlist.
- **R-CORS-004** MUST: CORS middleware configuration MUST use the 'origin' parameter rather than 'allowedHeaders' for origin validation.
- **R-CORS-005** SHOULD: Origin allowlist SHOULD be extracted to environment variables (e.g., CORS_ALLOWED_ORIGINS) with comma-separated values, parsed at application startup.
- **R-CORS-006** SHOULD: Separate configuration files for development and production environments SHOULD be implemented to ensure localhost origins are excluded from production builds.

### Verify

```bash
# Check for explicit CORS configuration with origin parameter
grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"

# Verify CORS headers are set correctly for allowed origins
curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"

# Check for localhost in CORS configuration (should not exist in production)
grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"
```

**Accept when:**
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins and requests from unauthorized origins are blocked
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol
- CORS middleware configuration uses the 'origin' parameter for origin validation, not 'allowedHeaders'
- Integration tests verify CORS behavior for both allowed and blocked origins

<enforcement>
Claude Code MUST NOT skip or defer verification. All CORS configuration changes MUST pass the verify commands before acceptance. Code review MUST validate CORS configuration against security requirements before merge. Security scanning tools MUST check for wildcard CORS configurations or missing origin validation.
</enforcement>