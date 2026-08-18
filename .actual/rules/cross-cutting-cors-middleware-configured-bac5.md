# Configure CORS Middleware with Explicit Origin Allowlist for Cross-Origin API Access: Cors Middleware Configured

These rules are ALWAYS ACTIVE for all Express.js HTTP endpoint configurations, CORS middleware initialization, and cross-origin request handling in the application.

### Rules

- **R-CORS-001** MUST: CORS middleware MUST be configured with an explicit allowlist of permitted origins rather than using wildcard (*) or permissive configurations.

### Verify

```bash
# Verify CORS middleware uses explicit origin allowlist (not wildcard)
grep -r "cors({" server/src/ | grep -E "(origin|allowedHeaders)" | grep -v "\*"

# Verify Access-Control-Allow-Origin headers are set correctly for allowed origins
curl -H "Origin: https://suraj-gov.github.io/sorter" -I http://localhost:${PORT:-3000}/ | grep -i "access-control-allow-origin"

# Check for localhost in CORS config (should not appear in production)
grep -r "localhost" server/src/ | grep -i cors && echo "WARNING: localhost found in CORS config"
```

**Accept when:**
- CORS middleware is configured with explicit origin allowlist (not wildcard) and registered before route handlers in the Express.js middleware stack
- Verification commands confirm that Access-Control-Allow-Origin headers are set correctly for allowed origins and requests from unauthorized origins are blocked
- Production configuration excludes development origins (localhost) and all allowed origins use HTTPS protocol
- The cors middleware configuration uses the 'origin' parameter rather than 'allowedHeaders' for origin validation
- Origin allowlist is extracted to environment variables (e.g., CORS_ALLOWED_ORIGINS) with comma-separated values
- Separate configuration files exist for development and production environments, with localhost origins excluded from production builds

<enforcement>
Claude Code MUST NOT skip or defer verification of CORS middleware configuration. All three verify commands MUST execute successfully before accepting changes to CORS middleware or origin allowlist configuration.
</enforcement>