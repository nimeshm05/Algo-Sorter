# Enforce Parameterized SQL Queries for Database Access: Sql Queries That

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SQL-001** MUST: All SQL queries that incorporate dynamic values MUST use parameterized queries with placeholder syntax ($1, $2, etc.) and separate values arrays.

### Verify

```bash
# Check for pool.query() calls without text: structure
grep -r "pool\.query(" server/src/ | grep -v "text:" | grep -v "SELECT SUM" | wc -l | grep -q "^0$"

# Check for template literals or string interpolation in SQL queries
grep -r "\${.*}" server/src/ | grep -i "select\|insert\|update\|delete" | wc -l | grep -q "^0$"

# Verify pool.query() calls use values: structure
grep -r "pool\.query({" server/src/ | grep "values:" | wc -l
```

**Accept when:**
- All pool.query() calls with dynamic values use the { text, values } object structure with placeholder syntax
- No SQL queries use string concatenation or template literals to incorporate user input
- Static analysis tools report zero SQL injection vulnerabilities in database access code

<enforcement>
Claude Code MUST NOT skip or defer verification. All database queries must be inspected for parameterization compliance before acceptance.
</enforcement>