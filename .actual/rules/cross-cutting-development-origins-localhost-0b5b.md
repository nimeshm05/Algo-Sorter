# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Development Origins Localhost

These rules are ALWAYS ACTIVE for all Express.js middleware configuration files, CORS setup code, and HTTP endpoint definitions that handle cross-origin requests in the application.

### Rules

- **R-CORS-001** MUST: Configure CORS middleware with an explicit origin allowlist (not wildcard) and register it before route handlers in the Express.js middleware stack.
- **R-CORS-002** MUST: Ensure production configuration excludes all development origins (localhost, 127.0.0.1, etc.) and all allowed origins use HTTPS protocol.
- **R-CORS-003** SHOULD: Development origins (localhost) SHOULD be included in the allowlist only when necessary for local testing workflows.
- **R-CORS-004** MUST: Use the 'origin' parameter in cors middleware configuration for origin validation, not 'allowedHeaders'.
- **R-CORS-005** SHOULD: Extract the origin allowlist to environment variables (e.g., CORS_ALLOWED_ORIGINS) with comma-separated values, parsed at application startup.
- **R-CORS-006** SHOULD: Implement separate configuration files for development and production environments to prevent accidental deployment of development origins.
- **R-CORS-007** MUST: Verify that Access-Control-Allow-Origin headers are correctly set for allowed origins and requests from unauthorized origins are blocked.

### Verify

```bash
# Check for explicit CORS configuration with origin parameter
grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"

# Verify CORS headers are set correctly for allowed origins
curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"

# Check for localhost in CORS configuration (should not exist in production)
grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"

# Verify no wildcard CORS policy is in use
grep -r "Access-Control-Allow-Origin.*\*" server/src/ && echo "ERROR: Wildcard CORS detected"
```

**Accept when:**
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins and requests from unauthorized origins are blocked
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol
- The cors middleware configuration uses the 'origin' parameter rather than 'allowedHeaders' for origin validation
- Origin allowlist is externalized to environment variables or configuration files, not hardcoded in application code
- Integration tests verify CORS behavior for both allowed and blocked origins

<enforcement>
Claude Code MUST NOT skip or defer verification of CORS configuration. All rules marked MUST are mandatory and must be verified before accepting any changes to CORS middleware or origin allowlist configuration. Code review and integration tests must confirm compliance before merge.
</enforcement>