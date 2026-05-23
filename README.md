# Web Security Labs — OWASP WebGoat

Hands-on documentation of [OWASP WebGoat](https://owasp.org/www-project-webgoat/) exercises: exploitation techniques, analysis, lessons learned, and mitigations aligned with the OWASP Top 10.

WebGoat is an intentionally vulnerable web application for learning web application security in a safe lab environment.

## Labs

| Module | OWASP category | Write-up |
| ------ | -------------- | -------- |
| [Broken Access Control](./WebGoat/Broken-Access-Control/WebGoat-Broken-Access-Control.md) | A01:2021 — Broken Access Control | IDOR, session hijacking, missing function-level access control, cookie spoofing |
| [SQL Injection (Intro)](./WebGoat/SQL-Injection/WebGoat-SQL-Injection.md) | A03:2021 — Injection | DML/DDL/DCL fundamentals, numeric and string SQLi, query chaining, CIA impact |

## Topics covered

- **Broken Access Control** — IDOR, predictable sessions, hidden admin endpoints, authentication cookie weaknesses
- **SQL Injection** — Query manipulation, authentication bypass, stacked queries, destructive SQL
- **Authentication & session management** — (planned)
- **Additional OWASP categories** — SSRF, XXE, insecure deserialization (planned)

## Repository structure

```
WEB-SECURITY-LABS/
├── README.md
└── WebGoat/
    ├── Broken-Access-Control/
    │   ├── WebGoat-Broken-Access-Control.md
    │   └── Screenshots/
    └── SQL-Injection/
        ├── WebGoat-SQL-Injection.md
        └── Screenshots/
```

Screenshots for each write-up live in that module's `Screenshots/` folder and are referenced with standard GitHub markdown image syntax.

## Disclaimer

These materials document **authorized security training** in isolated lab environments. Techniques described here must only be used on systems you own or have explicit permission to test.
