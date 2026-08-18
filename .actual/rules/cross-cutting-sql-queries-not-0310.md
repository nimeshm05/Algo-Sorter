# Enforce Parameterized SQL Queries for Database Access: Sql Queries Not

These rules are ALWAYS ACTIVE for all SQL queries executed via the pg library Pool.query() method, all database operations that incorporate request parameters, headers, or body data, and all SELECT, INSERT, UPDATE, and DELETE statements against any table in HTTP request handlers and middleware.

### Rules

- **R-SQL-001** MUST_NOT: SQL queries MUST NOT concatenate or interpolate user-controlled input directly into query strings.

### Verify

```bash
# Check for pool.query() calls without text: structure
grep -r "pool\.query(" server/src/ | grep -v "text:" | grep -v "SELECT SUM" | wc -l | grep -q "^0$"

# Check for template literals or string interpolation in SQL queries
grep -r "\${.*}" server/src/ | grep -i "select\|insert\|update\|delete" | wc -l | grep -q "^0$"

# Verify parameterized query structure adoption
grep -r "pool\.query({" server/src/ | grep "values:" | wc -l
```

**Accept when:**
- All pool.query() calls with dynamic values use the { text, values } object structure with placeholder syntax ($1, $2, $3)
- No SQL queries use string concatenation or template literals to incorporate user input
- Static analysis tools report zero SQL injection vulnerabilities in database access code
- Dynamic table or column names are validated against explicit allowlists before query construction

<enforcement>
Claude Code MUST NOT skip or defer verification. All database access points must be audited for compliance with parameterized query patterns before accepting pull requests or merging code.
</enforcement>