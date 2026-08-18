# Adopt Express Middleware Stack with CORS and JSON Parsing for HTTP Request Processing: Middleware Configuration Applied

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application exposes HTTP endpoints that require cross-origin resource sharing (CORS) to allow requests from specific frontend origins including https://suraj-gov.github.io/sorter and localhost:3000
- HTTP request bodies must be parsed as JSON to support client-server communication for visitor tracking operations
- The middleware stack processes all incoming requests before they reach route handlers, establishing security boundaries and data transformation layers
- The application implements a visitor tracking system that requires database access patterns (SELECT, INSERT, UPDATE) coordinated through middleware-configured request processing

## Problem Statement

HTTP services require a standardized approach to request preprocessing that enforces security policies (CORS), transforms request payloads (JSON parsing), and establishes consistent boundaries between external clients and internal data access logic. Without middleware-based request processing, each route handler would need to implement its own security checks and payload parsing, leading to inconsistent enforcement and duplicated code.

## Decision

1. MUST: Middleware configuration MUST be applied using app.use() before route definitions to ensure preprocessing occurs for all endpoints

## Policy Block

- MUST Middleware configuration MUST be applied using app.use() before route definitions to ensure preprocessing occurs for all endpoints

## Rationale

- The evidence shows explicit middleware configuration in server/src/index.ts with app.use(cors({allowedHeaders: [...]})) and app.use(express.json()), establishing a preprocessing layer before route handlers
- The detected pattern coordinates security.cors and boundaries.middleware facets, indicating that cross-origin security and request transformation are architectural concerns managed at the middleware boundary
- The application implements data access patterns (SELECT, INSERT, UPDATE) through route handlers that depend on preprocessed requests, demonstrating separation of concerns between request validation and data operations
- The pattern appears in a single file with 88.30% confidence, suggesting a centralized middleware configuration approach rather than distributed security enforcement

## Consequences

Positive:
- Centralized security policy enforcement through CORS middleware ensures consistent cross-origin access control across all endpoints
- Automatic JSON parsing eliminates boilerplate code in route handlers and reduces the risk of parsing errors or inconsistent payload handling
- Clear separation between request preprocessing (middleware) and business logic (route handlers) improves code maintainability and testability
- Middleware stack provides a single point of configuration for cross-cutting concerns like security, logging, and request transformation

Negative:
- Middleware execution order becomes critical; incorrect ordering can bypass security checks or cause parsing failures
- Global middleware applies to all routes, potentially adding overhead to endpoints that do not require JSON parsing or CORS
- CORS configuration using allowedHeaders instead of origin may not provide the intended security boundary, as allowedHeaders controls request headers rather than origins
- Debugging middleware-related issues requires understanding the entire middleware chain and execution order

## Alternatives

- Implement CORS and JSON parsing logic directly in each route handler (rejected)
  Rejected because: Duplicates security and parsing logic across multiple handlers, increasing maintenance burden and risk of inconsistent enforcement
  When valid: Only valid for applications with a single endpoint or when different endpoints require fundamentally different CORS policies
- Use a reverse proxy (nginx, API gateway) for CORS and request transformation (rejected)
  Rejected because: Adds infrastructure complexity and external dependencies; the evidence shows application-level middleware is sufficient for current requirements
  When valid: Valid for microservices architectures or when multiple backend services require unified CORS policies
- Use route-specific middleware instead of global app.use() configuration (rejected)
  Rejected because: The evidence shows all routes require CORS and JSON parsing; route-specific middleware would duplicate configuration without benefit
  When valid: Valid when different route groups require different middleware configurations or security policies

## Risks

- CORS misconfiguration using allowedHeaders instead of origin may not restrict cross-origin requests as intended, potentially exposing endpoints to unauthorized origins
  Mitigation: Review CORS configuration to use the 'origin' option instead of 'allowedHeaders' to properly restrict allowed origins; add integration tests to verify CORS behavior
  Owner: engineering team
- Middleware execution order changes during refactoring could bypass security checks or cause request processing failures
  Mitigation: Document required middleware order; implement automated tests that verify middleware execution sequence; use linting rules to detect middleware registration after route definitions
  Owner: engineering team
- Global JSON parsing middleware may cause issues with endpoints that expect non-JSON payloads (file uploads, webhooks)
  Mitigation: Monitor for parsing errors; consider route-specific middleware for endpoints with special payload requirements; document supported content types
  Owner: engineering team

## Implementation Notes

- Register middleware using app.use() before any route definitions (app.get, app.post, etc.) to ensure preprocessing occurs for all requests
- Verify CORS configuration uses the 'origin' option rather than 'allowedHeaders' to properly restrict cross-origin access: cors({ origin: ['https://suraj-gov.github.io/sorter', 'http://localhost:3000'] })
- Consider adding additional middleware for logging, error handling, and request validation to build a complete preprocessing pipeline
- Document the middleware execution order and the purpose of each middleware component for future maintainers

## Continuation Context


Verify commands:
- grep -n 'app\.use(cors(' server/src/index.ts
- grep -n 'app\.use(express\.json())' server/src/index.ts
- grep -B5 'app\.get\|app\.post' server/src/index.ts | grep -c 'app\.use'

Accept when:
- CORS middleware is configured using app.use(cors()) before route definitions
- JSON parsing middleware is configured using app.use(express.json()) before route definitions
- All middleware registrations appear before the first route handler definition in the source file

## Enforcement

- Verified by: Code review checklist verifying middleware configuration precedes route definitions
- Verified by: Automated grep-based verification in CI pipeline checking for app.use() patterns
- Verified by: Integration tests validating CORS headers and JSON parsing behavior
- Violation handling: CI pipeline fails if middleware patterns are not detected before route definitions
- Violation handling: Code review blocks merge if CORS or JSON parsing middleware is missing or misconfigured
- Violation handling: Runtime monitoring alerts on CORS-related errors or JSON parsing failures
- Exception process: Document exception rationale in ADR amendment if specific endpoints require different middleware configuration
- Exception process: Obtain architecture review approval for route-specific middleware that deviates from global configuration
- Exception process: Add inline comments explaining why standard middleware pattern is not followed