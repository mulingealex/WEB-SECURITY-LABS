# SQL Injection (Advanced) — UNION-Based Extraction — WebGoat

## Executive Summary

This lab demonstrates **UNION-based SQL injection**: manipulating a vulnerable query to merge attacker-controlled result sets with legitimate application output, then extracting sensitive credentials from a table that was never intended to be exposed through the search interface.

**Objective:** Exploit insufficient input handling on a last-name search feature to retrieve hidden authentication data from `user_system_data` via a crafted `UNION SELECT` payload.

**Impact demonstrated:** Confidentiality breach — plaintext credentials and session-related fields exfiltrated from a non-application-facing table.

---

## Lab Overview

The application exposes a **Search for User** function that filters records by last name. The feature queries the primary application table:

```sql
user_data
```

![WebGoat SQL Injection (Advanced) lab — user search interface and initial query context](./screenshots/Pasted%20image%2020260523154341.png)

*Figure 1 — Lab entry point: last-name search backed by the `user_data` table.*

During reconnaissance, schema enumeration revealed a second table not surfaced in normal application flow:

```sql
user_system_data
```

![Schema or query output revealing the hidden `user_system_data` table](./screenshots/Pasted%20image%2020260523154508.png)

*Figure 2 — Discovery of `user_system_data`, containing authentication-oriented columns.*

---

## Vulnerability Analysis

### Insecure Query Construction

The application likely builds SQL similar to:

```sql
SELECT * FROM user_data WHERE last_name = '$input'
```

![Application behavior or inferred query structure tied to the vulnerable last-name parameter](./screenshots/Pasted%20image%20202605231554807.png)

*Figure 3 — User input concatenated into the `WHERE` clause without parameterization.*

Because user input is embedded directly into the SQL statement—without prepared statements, strict typing, or allowlisting—the attacker can break out of the string context and append arbitrary SQL, including `UNION` clauses that pull data from other tables.

| Weakness | Security consequence |
| -------- | -------------------- |
| Dynamic string concatenation | Input becomes executable SQL syntax |
| No column/type validation on `UNION` branches | Cross-table data exfiltration |
| Over-privileged DB account | Access to tables beyond application scope |

---

## Exploitation Methodology

### Step 1 — Break Out of the SQL String

Close the opening quote on `last_name` to terminate the original string literal:

```sql
'
```

This shifts control to everything that follows in the query parser’s view of the statement.

---

### Step 2 — Introduce `UNION SELECT`

The `UNION` operator combines the result set of the original `SELECT` with a second `SELECT` chosen by the attacker:

```sql
' UNION SELECT ...
```

Both branches must be **union-compatible**: same number of columns and compatible data types per position.

---

### Step 3 — Determine Required Column Count

The legitimate query against `user_data` returns **seven** columns. Any injected `UNION SELECT` must therefore project exactly seven expressions.

#### `user_data` column layout

| # | Column |
| - | ------ |
| 1 | `userid` |
| 2 | `first_name` |
| 3 | `last_name` |
| 4 | `cc_number` |
| 5 | `cc_type` |
| 6 | `cookie` |
| 7 | `login_count` |

Column-count discovery techniques (e.g., `ORDER BY n` or incremental `UNION SELECT NULL,...`) apply when the schema is unknown; here, enumeration confirmed **7** columns upfront.

---

### Step 4 — Align Columns with `NULL` Padding

The target table `user_system_data` exposes four useful columns for exfiltration:

| # | Column |
| - | ------ |
| 1 | `userid` |
| 2 | `user_name` |
| 3 | `password` |
| 4 | `cookie` |

The remaining three positions required by the `UNION` are satisfied with `NULL` placeholders, which typically coerce cleanly across mixed types in many database engines:

```sql
userid, user_name, password, cookie, NULL, NULL, NULL
```

---

### Step 5 — Final Payload

Working injection submitted through the last-name parameter:

```sql
' UNION SELECT userid,user_name,password,cookie,NULL,NULL,NULL FROM user_system_data--
```

![Successful submission of the UNION-based payload in the WebGoat interface](./screenshots/Pasted%20image%2020260523154928.png)

*Figure 4 — Final payload: seven-column `UNION SELECT` against `user_system_data` with `NULL` padding.*

#### Payload breakdown

| Component | Role |
| --------- | ---- |
| `'` | Closes the original string literal on `last_name` |
| `UNION SELECT` | Appends an attacker-controlled result set |
| `userid, user_name, password, cookie` | Selects sensitive fields from the hidden table |
| `NULL, NULL, NULL` | Pads to seven columns to match `user_data` |
| `FROM user_system_data` | Targets the non-displayed credentials table |
| `--` | Line comment; neutralizes trailing syntax from the original query |

---

## Results

The injected query returned rows from `user_system_data` merged into the search results grid:

![Query results showing exfiltrated records from `user_system_data`](./screenshots/Pasted%20image%2020260523155028.png)

*Figure 5 — UNION output: usernames, passwords, and cookies from the hidden table visible in the UI.*

Among the recovered accounts, **Dave**’s credentials were identified:

| Field | Value |
| ----- | ----- |
| User | Dave |
| Password | `passW0rD` |

![Lab completion or credential confirmation for user Dave](./screenshots/Pasted%20image%2020260523155314.png)

*Figure 6 — Successful extraction and validation of Dave’s password (`passW0rD`).*

---

## Key Takeaways

1. **UNION injection** is a read-focused technique ideal when error messages are suppressed but result sets are rendered back to the user.
2. **Column count and type alignment** are mandatory; mismatch causes database errors and failed exfiltration.
3. **`NULL` padding** is a standard approach when the attacker needs fewer columns than the original query returns.
4. **Least privilege** and **parameterized queries** on the application tier remain the primary mitigations; exposing `user_system_data` to the same DB role as the search feature enabled this attack path.

---

## Mitigations (Defensive Reference)

| Control | Effect |
| ------- | ------ |
| Parameterized queries / prepared statements | User input cannot alter query structure |
| Strict allowlist validation on search input | Blocks quotes, comments, and SQL keywords |
| Principle of least privilege for DB users | Limits tables readable via injection |
| Output encoding and minimal data display | Reduces impact if injection occurs |
| WAF / query monitoring | Detects `UNION`, comment sequences, and anomalous `SELECT` patterns |

---

*Environment: OWASP WebGoat — SQL Injection (Advanced), UNION lesson. For authorized training and portfolio documentation only.*
