
# SQL Injection Exploit Demonstration (DVWA)

## Overview

This project demonstrates how to exploit an SQL Injection (SQLi) vulnerability in the DVWA (Damn Vulnerable Web Application) and provides guidance on how to patch and prevent such vulnerabilities.

## Vulnerable Path and Method

- **Vulnerable Path:** `/dvwa/vulnerabilities/sqli/`
- **HTTP Method:** `GET`
- **Vulnerable Parameter:** `id`
- **Vulnerability:** The `id` parameter is directly included in an SQL query without proper sanitization or parameterization, allowing attackers to inject arbitrary SQL code.

### Example Exploit

The following payload was used to exploit the vulnerability:

```
1' OR '1'='1
```

This payload, when submitted as the `id` parameter, returns all user records from the database, demonstrating a successful SQL Injection attack.

## Exploit Files

- `sql_injection_exploit.sh`: Bash script to automate the SQL Injection attack.
- Demonstration video and screenshots: See provided files in this repository.

## How to Patch SQL Injection Vulnerabilities

To prevent SQL Injection attacks, follow these best practices:

### 1. Use Prepared Statements (Parameterized Queries)

- Always use prepared statements with parameterized queries instead of directly embedding user input in SQL statements.

Example in PHP (MySQLi):

```php
$stmt = $mysqli->prepare("SELECT first_name, last_name FROM users WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
```

### 2. Use ORM Libraries

- Use Object-Relational Mapping (ORM) libraries that abstract SQL queries and automatically handle input sanitization.

### 3. Input Validation and Escaping

- Validate and sanitize all user inputs.
- Use built-in functions to escape special characters if parameterized queries are not available.

### 4. Least Privilege Principle

- Use database accounts with the minimum privileges necessary for the application.

### 5. Error Handling

- Do not expose detailed database errors to users. Use generic error messages.

### 6. Web Application Firewalls (WAF)

- Deploy a WAF to help detect and block SQL Injection attempts.

## References

- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [PHP: Prepared Statements](https://www.php.net/manual/en/mysqli.quickstart.prepared-statements.php)

---

**Note:** Always test your application for SQLi vulnerabilities and follow secure coding practices.
