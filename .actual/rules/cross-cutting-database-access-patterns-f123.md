# Enforce Parameterized SQL Queries for Database Access: Database Access Patterns

These rules are ALWAYS ACTIVE for all database access code using the pg library Pool.query() method, including all SQL queries that incorporate request parameters, headers, or body data in SELECT, INSERT, UPDATE, and DELETE statements.

### Rules

- **R-DB-001** SHOULD: Database access patterns SHOULD be reviewed during code review to verify parameterization is correctly applied.

### Verify

```bash
# Check for pool.query() calls without text: structure
grep -r "pool\.query(" server/src/ | grep -v "text:" | grep -v "SELECT SUM" | wc -l | grep -q "^0$"

# Check for template literals or string interpolation in SQL queries
grep -r "\${.*}" server/src/ | grep -i "select\|insert\|update\|delete" | wc -l | grep -q "^0$"

# Verify parameterized query structure is in use
grep -r "pool\.query({" server/src/ | grep "values:" | wc -l
```

**Accept when:**
- All pool.query() calls with dynamic values use the { text, values } object structure with placeholder syntax ($1, $2, $3)
- No SQL queries use string concatenation or template literals to incorporate user input
- Static analysis tools report zero SQL injection vulnerabilities in database access code
- Dynamic table or column names (if any) are validated against explicit allowlists before query construction

<enforcement>
Claude Code MUST NOT skip or defer verification. All database queries must be inspected to confirm parameterized query patterns are correctly applied. Violations detected by automated linting or static analysis MUST block pull requests until refactored to use parameterization.
</enforcement>