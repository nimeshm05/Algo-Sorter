# Enforce Parameterized SQL Queries for Database Access: Database Connection Pools

These rules are ALWAYS ACTIVE for all database access code using the pg library Pool.query() method, including all SQL queries executed against any table that incorporate request parameters, headers, or body data.

### Rules

- **R-DB-001** MUST: All database connection pools MUST be configured with appropriate SSL settings when connecting to remote databases.
- **R-DB-002** MUST: All database queries with dynamic values MUST use the parameterized query structure with { text, values } object format and placeholder syntax ($1, $2, $3).
- **R-DB-003** MUST: User-controlled input (request parameters, headers, body data) MUST NEVER be incorporated into SQL statements via string concatenation or template literals.
- **R-DB-004** MUST: All pool.query() calls MUST use numbered placeholders sequentially with corresponding entries in the values array in the same order.
- **R-DB-005** SHOULD: Dynamic table or column names SHOULD be validated against an explicit allowlist of permitted identifiers before query construction.
- **R-DB-006** SHOULD: ESLint or custom linting rules SHOULD be configured to flag string concatenation or template literals containing SQL keywords and user input variables.

### Verify

```bash
# Verify no pool.query() calls bypass parameterization
grep -r "pool\.query(" server/src/ | grep -v "text:" | grep -v "SELECT SUM" | wc -l | grep -q "^0$"

# Verify no template literals with SQL keywords and user input
grep -r "\${.*}" server/src/ | grep -i "select\|insert\|update\|delete" | wc -l | grep -q "^0$"

# Verify parameterized queries with values arrays are in use
grep -r "pool\.query({" server/src/ | grep "values:" | wc -l
```

**Accept when:**
- All pool.query() calls with dynamic values use the { text, values } object structure with placeholder syntax ($1, $2, $3)
- No SQL queries use string concatenation or template literals to incorporate user input
- Static analysis tools report zero SQL injection vulnerabilities in database access code
- All database operations in HTTP request handlers and middleware follow parameterized query patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. All database queries must be inspected for parameterization compliance before acceptance.
</enforcement>