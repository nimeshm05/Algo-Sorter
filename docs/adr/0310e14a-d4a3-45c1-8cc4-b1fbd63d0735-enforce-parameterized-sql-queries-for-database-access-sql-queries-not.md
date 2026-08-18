# Enforce Parameterized SQL Queries for Database Access: Sql Queries Not

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The application uses PostgreSQL as its primary datastore via the pg library, executing SELECT, INSERT, and UPDATE operations against a visitors table
- Database queries handle user-controlled input (req.ip) that must be safely incorporated into SQL statements to prevent injection attacks
- The codebase demonstrates a pattern of using parameterized queries with placeholder syntax ($1, $2, $3) and separate values arrays for all database operations involving dynamic data
- The application exposes HTTP endpoints that trigger database operations, creating a direct path from external input to SQL execution

## Problem Statement

Without consistent use of parameterized queries, applications that incorporate user-controlled input into SQL statements are vulnerable to SQL injection attacks, which can lead to unauthorized data access, data manipulation, or complete database compromise. The pattern must be enforced across all database access points to maintain security posture.

## Decision

1. MUST_NOT: SQL queries MUST NOT concatenate or interpolate user-controlled input directly into query strings

## Policy Block

- MUST_NOT SQL queries MUST NOT concatenate or interpolate user-controlled input directly into query strings

In scope:
- All SQL queries executed via pg library Pool.query() method
- All database operations that incorporate request parameters, headers, or body data
- SELECT, INSERT, UPDATE, and DELETE statements against any table
- Database queries in HTTP request handlers and middleware

Out of scope:
- Static SQL queries with no dynamic values (e.g., schema migrations, static aggregations)
- Database connection configuration and pool initialization
- ORM-generated queries that handle parameterization internally

Exceptions:
- EXC-001: Static queries with zero dynamic values that are verified to contain no user input

## Rationale

- The evidence shows consistent use of parameterized queries across all database operations in server/src/index.ts, with pool.query() calls using text/values structure for SELECT, INSERT, and UPDATE operations
- Parameterized queries provide the strongest defense against SQL injection by ensuring user input is treated as data rather than executable SQL code, regardless of input content
- The pg library's native support for parameterized queries with placeholder syntax provides a straightforward, performant mechanism that requires no additional dependencies
- The pattern is already established in the codebase with 100% adoption in the detected file, indicating feasibility and team familiarity

## Consequences

Positive:
- Eliminates SQL injection vulnerabilities by preventing user input from being interpreted as SQL commands
- Improves query performance through database query plan caching for parameterized statements
- Provides clear, auditable code patterns that are easily verified during security reviews
- Maintains compatibility with PostgreSQL prepared statement optimization

Negative:
- Requires developers to structure queries using text/values objects rather than simple string templates
- May increase initial development time for developers unfamiliar with parameterized query syntax
- Cannot use parameterization for dynamic table or column names, requiring alternative validation approaches for those cases
- Adds slight verbosity to query code compared to string concatenation

## Alternatives

- Use an ORM (e.g., TypeORM, Sequelize) that handles parameterization automatically (rejected)
  Rejected because: Introduces significant additional complexity and dependencies; current pg library usage is lightweight and sufficient for the application's needs
  When valid: For applications with complex data models requiring extensive relationship management and migrations
- Implement manual input sanitization and escaping before query execution (rejected)
  Rejected because: Manual sanitization is error-prone and provides weaker security guarantees than parameterized queries; requires maintaining custom escaping logic
  When valid: Never recommended; parameterized queries are the industry standard
- Use stored procedures for all database operations (deferred)
  Rejected because: Not rejected but deferred; would provide additional security layer but requires significant refactoring
  When valid: For applications with complex business logic that benefits from database-side execution and additional access control layers

## Risks

- Developers may inadvertently introduce string concatenation in new code, bypassing parameterization
  Mitigation: Implement automated linting rules to detect SQL string concatenation patterns; enforce code review checklist items for database queries
  Owner: Engineering team and security reviewers
- Dynamic table or column names cannot be parameterized, potentially leading to inconsistent security patterns
  Mitigation: Establish separate validation rules for dynamic identifiers using allowlists; document exceptions clearly
  Owner: Engineering team
- Legacy code or third-party integrations may not follow parameterization patterns
  Mitigation: Conduct security audit of all database access points; create migration plan for non-compliant code
  Owner: Security team and engineering leads

## Implementation Notes

- Use pool.query({ text: 'SELECT * FROM table WHERE column = $1', values: [userInput] }) structure for all queries with dynamic values
- Number placeholders sequentially ($1, $2, $3) and ensure the values array contains corresponding entries in the same order
- For dynamic table or column names, validate against an explicit allowlist of permitted identifiers before query construction
- Configure ESLint or custom linting rules to flag string concatenation or template literals containing SQL keywords and user input variables

## Continuation Context


Verify commands:
- grep -r "pool\.query(" server/src/ | grep -v "text:" | grep -v "SELECT SUM" | wc -l | grep -q "^0$"
- grep -r "\${.*}" server/src/ | grep -i "select\|insert\|update\|delete" | wc -l | grep -q "^0$"
- grep -r "pool\.query({" server/src/ | grep "values:" | wc -l

Accept when:
- All pool.query() calls with dynamic values use the { text, values } object structure with placeholder syntax
- No SQL queries use string concatenation or template literals to incorporate user input
- Static analysis tools report zero SQL injection vulnerabilities in database access code

## Enforcement

- Verified by: Automated static analysis via ESLint rules detecting SQL string concatenation patterns
- Verified by: Code review checklist requiring verification of parameterized queries for all database operations
- Verified by: Security scanning tools (e.g., Semgrep, CodeQL) configured to detect SQL injection patterns
- Violation handling: CI pipeline fails if linting rules detect non-parameterized SQL queries with dynamic values
- Violation handling: Pull requests containing SQL string concatenation are blocked until refactored to use parameterization
- Violation handling: Security team is notified of violations detected in production code for immediate remediation
- Exception process: Developer documents the specific case requiring an exception with technical justification
- Exception process: Security team reviews the exception request and validates that alternative controls are in place
- Exception process: Approved exceptions are documented in code comments with EXC-ID reference and expiration review date