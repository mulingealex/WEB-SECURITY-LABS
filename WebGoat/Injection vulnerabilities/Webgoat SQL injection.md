A SQL injection attack consists of insertion or "injection" of malicious code via the SQL query input from the client to the application.

SQL injections can occur when unfiltered data from the client, such as input from a search field, gets into the SQL interpreter of the application itself. If an application fails to either correctly sanitize user input (using prepared statements or similar) or filter the input against special characters, hackers can manipulate the underlying SQL statement to their advantage.

Throughout the SQL Injection project, this table becomes the target used to understand how applications interact with databases and how attackers manipulate SQL queries. 

![[Pasted image 20260522133814.png]]

Initially, we use normal SQL queries to retrieve legitimate data, such as finding Bob Franco’s department. As the lessons progress, the same table helps demonstrate how insecure input handling can allow attackers to alter query logic, bypass filters, extract sensitive information, modify records, or access unauthorized data.

The general flow is:

```
User Input → Application Query → Employees Table → Database Response
```

In secure applications:

- user input is treated only as data

In vulnerable applications:

- user input can become executable SQL commands

This table will therefore be used to:

- practice normal SQL queries
- understand how databases process requests
- test SQL Injection payloads
- observe how injected conditions affect returned results
- analyze the impact on Confidentiality, Integrity, and Availability (CIA triad)

As the project advances, you’ll move from simple data retrieval to authentication bypass, query manipulation, UNION injections, and eventually more advanced SQLi techniques using this same database interaction model.

Executed a structured SQL query within the WebGoat SQL Injection Intro module to retrieve employee department information from the `employees` table. 

![[Pasted image 20260522133119.png]]

This  demonstrated how SQL queries interact with relational databases using the `SELECT`, `FROM`, and `WHERE` clauses to filter and return specific records. This lab builds foundational understanding of database operations, query logic, and how improper input handling can later lead to SQL Injection vulnerabilities.

This section introduces the three primary categories of SQL commands used in database management and commonly referenced in SQL Injection security testing:

• Data Manipulation Language (DML) — Used to retrieve and modify data within database tables through commands such as SELECT, INSERT, UPDATE, and DELETE.

• Data Definition Language (DDL) — Used to define or alter database structures using commands like CREATE, ALTER, and DROP.

• Data Control Language (DCL) — Used to manage database permissions and access control through commands such as GRANT and REVOKE.

Understanding these SQL categories is essential for recognizing how SQL Injection attacks can manipulate database operations and impact the Confidentiality, Integrity, and Availability (CIA) of systems.

## 1. Data Manipulation Language(DML)

This section introduces Data Manipulation Language (DML), a category of SQL commands used to interact with and manage data stored within relational databases. The lesson covers core DML operations including SELECT, INSERT, UPDATE, and DELETE, which are responsible for retrieving, adding, modifying, and removing records from database tables.

![[Pasted image 20260522140418.png]]

Completed a Data Manipulation Language (DML) exercise in WebGoat by using an UPDATE statement to modify employee department information within the employees table. The lab demonstrated how SQL queries can alter existing database records using the UPDATE, SET, and WHERE clauses, while highlighting how insecure query handling could allow attackers to manipulate data and compromise system integrity through SQL Injection vulnerabilities.

## 2. Data Definition Language

Data definition language includes commands for defining data structures. DDL commands are commonly used to define a database’s schema. The schema refers to the overall structure or organization of the database and. in SQL databases, includes objects such as tables, indexes, views, relationships, triggers, and more.

If an attacker successfully "injects" DDL type SQL commands into a database, he can violate the integrity (using ALTER and DROP statements) and availability (using DROP statements) of a system.

- DDL commands are used for creating, modifying, and dropping the structure of database objects.
    
- CREATE - create database objects such as tables and views
    
- ALTER - alters the structure of the existing database
    
- DROP - delete objects from the database

![[Pasted image 20260522142057.png]]

Completed the WebGoat Data Definition Language (DDL) module by exploring how SQL commands define and modify database structures. 
Practiced using CREATE, ALTER, and DROP statements to understand schema management and successfully modified the employees table by adding a new column using the ALTER TABLE command. 
The exercise also demonstrated how insecure handling of DDL operations in vulnerable applications can allow attackers to manipulate or destroy database structures, impacting system integrity and availability.

## 3. Data Control Language(DCL)

Data control language is used to implement access control logic in a database. DCL can be used to revoke and grant user privileges on database objects such as tables, views, and functions.

If an attacker successfully "injects" DCL type SQL commands into a database, he can violate the confidentiality (using GRANT commands) and availability (using REVOKE commands) of a system. For example, the attacker could grant himself admin privileges on the database or revoke the privileges of the true administrator.

- DCL commands are used to implement access control on database objects.
    
- GRANT - give a user access privileges on database objects
    
- REVOKE - withdraw user privileges that were previously given using GRANT

![[Pasted image 20260522143400.png]]

Completed the WebGoat Data Control Language (DCL) module by exploring database access control and permission management using GRANT and REVOKE statements. 

### SQL INJECTION AND THE CIA TRIAD

SQL Injection (SQLi) attacks directly impact the CIA Triad by allowing attackers to manipulate insecure database queries and perform unauthorized actions on backend databases and applications.

• Confidentiality — Attackers can retrieve sensitive information such as usernames, passwords, credit card details, and personal records using injected SELECT queries.

• Integrity — Attackers can modify or delete existing data using malicious UPDATE or DELETE statements, compromising the accuracy and trustworthiness of database records.

• Availability — Attackers can disrupt services or destroy database structures using commands such as DROP TABLE, potentially causing application failures or denial-of-service conditions.

Understanding how SQL Injection affects the CIA triad is essential for analyzing the real-world security impact of insecure database query handling.

### Numeric SQL Injection

After understanding how SQL Injection impacts the CIA triad, the next section introduces Numeric SQL Injection, a technique that targets applications using unsanitized numeric input within SQL queries. Unlike string-based SQL Injection, numeric SQLi does not require quotation marks because the application directly inserts numerical values into the query structure.

This vulnerability occurs when user-controlled numeric parameters are dynamically concatenated into SQL statements without proper validation, allowing attackers to manipulate query logic using conditions such as `OR 1=1`. Successful exploitation can expose sensitive records, bypass filtering conditions, and compromise the confidentiality of backend databases.

![[Pasted image 20260522151348.png]]

## 1.Compromising confidentiality with String SQL injection

This lab demonstrates a string-based SQL Injection vulnerability within an internal employee management system. The application requires employees to provide their last name and an authentication TAN (Transaction Authentication Number) to retrieve personal department and salary information from the backend database.

The objective of the exercise was to exploit insecure SQL query construction to bypass authentication controls and retrieve confidential employee records from the `employees` table without knowing valid authentication credentials for other users.

![[Pasted image 20260522160333.png]]
## Vulnerable Query Structure
The application dynamically constructed the following SQL query:

```
SELECT * FROM employeesWHERE last_name = '" + name + "'AND auth_tan = '" + auth_tan + "'";
```

Because user input was concatenated directly into the SQL statement without proper sanitization or parameterized queries, the application became vulnerable to SQL Injection.
## Injection Payload
The following payload was injected into the Authentication TAN field:

```
3SL99A' OR '1'='1
```

Resulting SQL query:

```
SELECT * FROM employeesWHERE last_name='Smith'AND auth_tan='3SL99A' OR '1'='1';
```

![[Pasted image 20260522153927.png]]
## Exploitation Analysis

The injected condition:

```
OR '1'='1'
```

always evaluates to TRUE. This manipulated the query logic and caused the database to return all employee records from the `employees` table instead of restricting access to a single authenticated user.

The attack successfully bypassed the intended authorization mechanism and exposed sensitive internal employee information,
## 2. Compromising Integrity with Query chaining

This lab demonstrates how SQL Injection can be used to compromise the integrity of a database through SQL query chaining (stacked queries). The vulnerable application dynamically constructed SQL statements using unsanitized user input, allowing additional SQL commands to be appended and executed within the same request.

The objective of the exercise was to manipulate the employee salary records by injecting an unauthorized `UPDATE` statement into the backend query.

![[Pasted image 20260522155926.png]]
## Vulnerable Query Concept
The application accepted:
- Employee Name
- Authentication TAN
and dynamically concatenated the values into a SQL query without using parameterized statements.

This insecure query handling enabled stacked query execution using the semicolon (`;`) metacharacter to terminate the original statement and append a second malicious query.

## Injection Payload

Injected payload:

```
Smith'; UPDATE employeesSET salary=999999WHERE last_name='Smith
```

Authentication TAN:

```
3SL99A
```


![[Pasted image 20260522160118.png]]

## Exploitation Result

The injected `UPDATE` statement successfully modified John Smith’s salary within the `employees` table, making the attacker-controlled account the highest-paid employee in the database.

The attack demonstrated how SQL Injection can move beyond information disclosure and directly manipulate database records through chained SQL statements.

## 3.Compromising Availability

This lab demonstrates how SQL Injection can be used to compromise the Availability component of the CIA triad through destructive query chaining techniques. The vulnerable application dynamically constructed SQL queries using unsanitized user input, allowing attackers to inject additional SQL statements into backend database operations.

The objective of the exercise was to delete the `access_log` table, which stored records of employee activity within the system.
## Vulnerable Query Concept

The application accepted user-controlled input within a search field that was likely incorporated into a SQL query similar to:

```
SELECT * FROM access_logWHERE action LIKE '%INPUT%';
```

![[Pasted image 20260522162340.png]]

Because the input was concatenated directly into the SQL statement without parameterized queries or input sanitization, the application became vulnerable to stacked-query SQL Injection.
## Injection Payload
Injected payload:

```
'; DROP TABLE access_log;--
```
## Payload Breakdown

| Payload Component       | Purpose                               |
| ----------------------- | ------------------------------------- |
| `'`                     | Closes the original SQL string        |
| `;`                     | Terminates the original SQL statement |
| `DROP TABLE access_log` | Executes a destructive SQL query      |
| `--`                    | Comments out remaining SQL syntax     |
![[Pasted image 20260522162445.png]]

## Exploitation Result
The injected query successfully deleted the `access_log` table from the database. As a result:
- activity logs became inaccessible
- logging functionality was disrupted
- database resources became unavailable to legitimate users and administrators

This demonstrated how SQL Injection vulnerabilities can escalate from data disclosure to destructive database operations.

## Conclusion — SQL Injection (Intro)

The SQL Injection Intro module demonstrated how insecure handling of user-controlled input can allow attackers to manipulate backend SQL queries and compromise database security. Through hands-on exercises, the lab explored fundamental SQL concepts, including DML, DDL, and DCL operations, while showing how SQL Injection vulnerabilities can impact the Confidentiality, Integrity, and Availability (CIA) of systems.

The exercises covered both string-based and numeric SQL Injection techniques, authentication bypass, query chaining, and destructive SQL operations such as unauthorized data modification and table deletion. These scenarios highlighted how dynamically constructed SQL queries without proper input validation or parameterized statements can lead to unauthorized data access, privilege escalation, integrity compromise, and disruption of critical database services.

Overall, the module reinforced the importance of secure database interaction practices, including parameterized queries, prepared statements, input validation, least privilege access control, and secure query handling to mitigate SQL Injection vulnerabilities in modern applications.





