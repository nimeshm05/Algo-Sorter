# Define Express HTTP Service Boundaries with CORS and Route Handlers: Http Service Boundaries

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application exposes HTTP endpoints using Express framework with explicit route definitions for visitor tracking functionality
- CORS middleware is configured to restrict cross-origin requests to specific allowed origins (https://suraj-gov.github.io/sorter and localhost:3000)
- Service boundaries are established through route handlers that coordinate database operations with PostgreSQL connection pooling
- The application requires secure handling of database credentials via environment variables and SSL-enforced connections
- Runtime configuration is sourced from process environment variables for DATABASE_URL and PORT

## Problem Statement

Without explicit service boundary definitions and CORS policies, HTTP services risk exposing endpoints to unauthorized origins, lack clear separation between routing logic and data access patterns, and may inadvertently leak sensitive configuration or allow uncontrolled cross-origin access.

## Decision

1. MUST: HTTP service boundaries MUST be defined using Express route handlers with explicit path definitions (e.g., app.get('/', handler))

## Policy Block

- MUST HTTP service boundaries MUST be defined using Express route handlers with explicit path definitions (e.g., app.get('/', handler))

In scope:
- Express-based HTTP services with route handlers
- Services requiring CORS configuration for browser-based clients
- Applications using PostgreSQL connection pooling with environment-based configuration
- Endpoints that coordinate database operations with HTTP responses

Out of scope:
- GraphQL or gRPC service definitions
- WebSocket or real-time streaming endpoints beyond HTTP request/response
- Static file serving or CDN-backed content delivery
- Internal service-to-service communication without browser CORS requirements

Exceptions:
- EXC-001: Development or testing environments require wildcard CORS origins
- EXC-002: Legacy endpoints require migration period with relaxed CORS policies

## Rationale

- Evidence shows explicit CORS configuration with allowedHeaders restricting access to two specific origins, demonstrating security-conscious boundary enforcement
- Route handlers (app.get('/', ...), app.get('/hello', ...)) establish clear service boundaries with coordinated database operations using parameterized queries
- SSL-enforced PostgreSQL connections and environment-based secrets handling (process.env.DATABASE_URL) indicate secure coding practices for data access
- The pattern of middleware registration (app.use(cors(...)), app.use(express.json())) before route definitions ensures proper request processing pipeline

## Consequences

Positive:
- Explicit CORS policies prevent unauthorized cross-origin access from untrusted domains
- Clear service boundaries through route handlers improve code organization and maintainability
- Environment-based configuration enables secure deployment across environments without code changes
- SSL-enforced database connections protect data in transit between application and PostgreSQL

Negative:
- CORS allowedHeaders configuration requires maintenance when adding new approved origins
- SSL with rejectUnauthorized: false may accept invalid certificates, reducing security guarantees
- Tight coupling between route handlers and database pool queries reduces testability without mocking
- Environment variable dependencies create runtime configuration complexity and potential startup failures

## Alternatives

- Use API Gateway or reverse proxy (e.g., nginx, Kong) for CORS and routing instead of application-level middleware (rejected)
  Rejected because: Evidence shows application-level CORS configuration already implemented; infrastructure-level approach would require significant refactoring and deployment changes
  When valid: Valid for microservices architectures with centralized API gateway requirements or when multiple services need consistent CORS policies
- Implement GraphQL or tRPC for type-safe service boundaries instead of REST endpoints (rejected)
  Rejected because: Current implementation uses Express REST patterns with simple visitor tracking; GraphQL overhead not justified by complexity
  When valid: Valid for complex APIs with multiple clients requiring flexible query capabilities or strong type safety requirements
- Use framework-agnostic HTTP server (e.g., Node.js http module) without Express middleware (rejected)
  Rejected because: Express provides established patterns for middleware composition and routing; raw HTTP server would require reimplementing CORS and JSON parsing
  When valid: Valid for performance-critical services with minimal routing needs or when minimizing dependency footprint is critical

## Risks

- CORS allowedHeaders misconfiguration could block legitimate clients or allow unauthorized origins
  Mitigation: Implement automated tests verifying CORS headers for approved and blocked origins; document origin approval process
  Owner: Security team and backend engineering
- SSL rejectUnauthorized: false setting may accept man-in-the-middle attacks with invalid certificates
  Mitigation: Evaluate cloud provider certificate validity; consider using proper CA certificates or connection string SSL parameters
  Owner: Infrastructure and security teams
- Missing environment variables (DATABASE_URL, PORT) cause runtime failures without clear error messages
  Mitigation: Implement startup validation checking required environment variables; provide clear error messages with configuration guidance
  Owner: Backend engineering team

## Implementation Notes

- Register CORS middleware before route handlers using app.use(cors({allowedHeaders: [...]})) to ensure proper request filtering
- Use parameterized queries (pool.query({text: '...', values: [...]})) for all database operations to prevent SQL injection
- Configure PostgreSQL connection pool with SSL settings: {connectionString: process.env.DATABASE_URL, ssl: {rejectUnauthorized: false}}
- Structure route handlers with async/await and send JSON responses using res.send({...}) for consistent API contracts
- Validate environment variables at application startup before initializing database connections or starting HTTP server

## Continuation Context


Verify commands:
- grep -r 'app.use(cors' server/src/ && grep -r 'allowedHeaders' server/src/
- grep -r 'process.env.DATABASE_URL' server/src/ && grep -r 'ssl:' server/src/
- grep -r 'app.get\|app.post\|app.put\|app.delete' server/src/ | grep -v node_modules

Accept when:
- CORS middleware is configured with explicit allowedHeaders array before route definitions
- All database connection strings are sourced from process.env variables with SSL configuration
- Route handlers use Express app.get/post/put/delete methods with explicit path strings and async handlers

## Enforcement

- Verified by: Code review checklist verifying CORS configuration and environment variable usage
- Verified by: Static analysis scanning for hardcoded credentials or connection strings
- Verified by: Integration tests validating CORS headers for approved and blocked origins
- Verified by: Security scanning tools checking SSL configuration and secrets handling
- Violation handling: CI pipeline fails if hardcoded credentials or connection strings detected
- Violation handling: Code review blocks merge if CORS configuration missing or misconfigured
- Violation handling: Security team notified for violations involving SSL or secrets handling
- Violation handling: Automated remediation suggestions provided for common violations
- Exception process: Submit exception request to security team with justification and risk assessment
- Exception process: Architecture review required for deviations from Express routing patterns
- Exception process: Time-bound exceptions must include sunset date and migration plan
- Exception process: All exceptions documented in security exception registry with approval trail