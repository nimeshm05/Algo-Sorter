# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Allowlist Not Include

These rules are ALWAYS ACTIVE for all HTTP endpoints exposed by the Express.js application, cross-origin requests from browser-based clients, and middleware stack initialization in server/src/index.ts.

### Rules

- **R-CORS-001** MUST_NOT: The allowlist MUST NOT include origins that are not under organizational control or explicitly trusted.

### Verify

```bash
# Check for explicit CORS configuration with origin parameter (not wildcard)
grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"

# Verify Access-Control-Allow-Origin headers are set correctly for allowed origins
curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"

# Check for localhost in CORS config (should not be in production)
grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"
```

**Accept when:**
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins and requests from unauthorized origins are blocked
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol
- The cors middleware configuration uses the 'origin' parameter rather than 'allowedHeaders' for origin validation
- Origin allowlist is extracted to environment variables (e.g., CORS_ALLOWED_ORIGINS) with comma-separated values
- Separate configuration files exist for development and production environments, with localhost origins excluded from production builds
- Integration tests verify CORS headers are correctly set for allowed origins and blocked for unauthorized origins

<enforcement>
Claude Code MUST NOT skip or defer verification of CORS configuration. All verification commands MUST be executed before accepting changes to CORS middleware. Code review MUST validate CORS configuration against security requirements. Security team MUST review any changes to CORS allowlist or middleware configuration.
</enforcement>