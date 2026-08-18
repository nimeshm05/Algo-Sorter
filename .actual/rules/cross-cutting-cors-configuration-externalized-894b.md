# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Cors Configuration Externalized

These rules are ALWAYS ACTIVE for all Express.js HTTP endpoint configurations, CORS middleware initialization, and cross-origin request handling in the application.

### Rules

- **R-CORS-001** SHOULD: CORS configuration SHOULD be externalized to environment-specific configuration files rather than hardcoded in application logic.
- **R-CORS-002** MUST: CORS middleware MUST use explicit origin allowlist (not wildcard) and be registered before route handlers in the Express.js middleware stack.
- **R-CORS-003** MUST: Production CORS configuration MUST exclude development origins (localhost) and all allowed origins MUST use HTTPS protocol.
- **R-CORS-004** MUST: CORS middleware configuration MUST use the 'origin' parameter rather than 'allowedHeaders' for origin validation.
- **R-CORS-005** SHOULD: Origin allowlist SHOULD be extracted to environment variables (e.g., CORS_ALLOWED_ORIGINS) with comma-separated values, parsed at application startup.
- **R-CORS-006** SHOULD: Separate configuration files SHOULD be implemented for development and production environments.
- **R-CORS-007** MAY: Public API endpoints explicitly designed for third-party consumption MAY use more permissive CORS policies (EXC-001).

### Verify

```bash
# Verify CORS middleware uses 'origin' parameter, not 'allowedHeaders'
grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"

# Verify Access-Control-Allow-Origin headers are set correctly for allowed origins
curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"

# Verify localhost is not present in production CORS configuration
grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"
```

**Accept when:**
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins and requests from unauthorized origins are blocked
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol
- CORS middleware configuration uses the 'origin' parameter rather than 'allowedHeaders' for origin validation
- Origin allowlist is externalized to environment variables or configuration files, not hardcoded in application logic

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review process MUST validate CORS configuration against security requirements before merge. Integration tests MUST verify CORS behavior for both allowed and blocked origins in CI pipeline. Security scanning tools MUST check for wildcard CORS configurations or missing origin validation. Manual security review MUST be conducted for any changes to CORS middleware configuration or allowlist.
</enforcement>