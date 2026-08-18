# Enforce Parameterized SQL Queries for Database Access: Database Query Operations

These rules are ALWAYS ACTIVE for all database query operations in files matching the configured scope, particularly those handling external input through HTTP request parameters, headers, or body data.

### Rules

- **R-DB-001** MUST: Database query operations (SELECT, INSERT, UPDATE, DELETE) that accept external input MUST pass parameters through the values array of the query configuration object, using placeholder syntax ($1, $2, $3) with corresponding entries in the values array in the same order.

### Verify

```bash
# Check for pool.query() calls without text: property
grep -r "pool\.query(" server/src/ | grep -v "text:" | grep -v "SELECT SUM" | wc -l | grep -q "^0$"

# Check for template literals or string interpolation in SQL queries
grep -r "\${.*}" server/src/ | grep -i "select\|insert\|update\|delete" | wc -l | grep -q "^0$"

# Verify pool.query() calls use { text, values } structure
grep -r "pool\.query({" server/src/ | grep "values:" | wc -l
```

**Accept when:**
- All pool.query() calls with dynamic values use the { text, values } object structure with placeholder syntax ($1, $2, $3)
- No SQL queries use string concatenation or template literals to incorporate user input
- Static analysis tools report zero SQL injection vulnerabilities in database access code
- Static queries with zero dynamic values containing no user input are permitted as exceptions (EXC-001)

<enforcement>
Claude Code MUST NOT skip or defer verification. All database query operations must be inspected for parameterization compliance before accepting changes.
</enforcement>