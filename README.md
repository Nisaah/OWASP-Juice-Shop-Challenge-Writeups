# OWASP-Juice-Shop-Pentest
Penetration test report for OWASP Juice Shop covering common OWASP Top 10 vulnerabilities, POC screenshots, and remediation steps

## Objective
Practice web application penetration testing in a safe environment

## Scope
- Target: OWASP Juice shop (intentionally vulnerable web application)
https://juice-shop.herokuapp.com/#/
- Testing Type: Black box web application penetration testing
  
## Tools Used
- Burpsuite
- Nmap
- SQLMap
- Firefox Developer tools

## Contents
- Owasp-juice-shop-pentest-report.pdf -> full report with findings, poc screenshots, and remediation
- findings -> Individual vulnerability write ups

## Findings
## 1. SQL Injection Login Bypass
### Explanation
The application is vulnerable to SQL Injection (OWASP Top 10: A03 - Injection). User input is inserted directly into an SQL query without proper validation or escaping. This allows an attacker to modify the query logic by injecting SQL code. In this case, using ' OR 1=1; in the username field makes the condition always true, causing the database to return the first user (usually the administrator), allowing unauthorized login regardless of the password entered.

### Exploit Steps
- Navigate to the login page of juice shop, https://juice-shop.herokuapp.com/#/login
  I tested the login form with the following credentials:
  **Username**: admin'or1=1;
  **Password**: enter any random values
I was successfully logged in as the **admin user** (admin@juice-sh.op), without needing to know the emai and password.
This is a classic **SQL Injection** vulnerability. The injected input modified the SQL query behind the login form to:

### POC

### Security Impact

- Full admin access without authentication
- Bypass of authorization controls
- High risk if in production: could lead to full data exposure or manipulation
### OWASP Mapping
- A01:2021 – Broken Access Control
- A03:2021 – Injection
### Remediations
- Use parameterized queries
- Use an ORM or query builder: Tools like Sequelize, Hibernate, or Entity Framework automatically protect against SQL injection.
- Validate all user input
- Don’t show detailed error messages
## 1. Accessing Hidden Files via robots.txt and Null Byte Injection in Juice Shop
### Explanation
robots.txt Discovery
While exploring the application, accessed the /robots.txt file: https://juice-shop.herokuapp.com/robots.txt
and saw:
<pre>User-agent: *
Disallow: /ftp</pre>
This told us that the /ftp directory is intentionally hidden from web crawlers.
#### Exploring /ftp
Since /ftp was disallowed, it hinted that there might be something interesting there.
<pre>https://juice-shop.herokuapp.com/ftp</pre>
showed a list of files, including acquisitions.md,eastere.gg,announcement_encrypted.md, legal.md, etc
#### Access restriction on eastere.gg
Attempting to access the file directly:
<pre>https://juice-shop.herokuapp.com/ftp/eastere.gg</pre>
resulted in a 403 Forbidden error with the message:
<pre>Only .md and .pdf files are allowed!</pre>
The application had a security filter restricting access to files by extension, allowing only .md and .pdf files.
#### Bypassing with Null Byte Injection
Using Null Byte Injection — a technique where a special %00 (null byte) character is inserted in the filename, allows bypassing the file extension check.
We accessed:
<pre>https://juice-shop.herokuapp.com/ftp/eastere.gg%2500.md</pre>
where %2500 is the URL-encoded form of %00 (null byte).
This tricks the filter into thinking the filename ends with .md, passing the check, but the system reads it as eastere.gg.

Congratulations, you found the easter egg!
The real easter egg can be found here:
L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==

This base64 string can be decoded to get the next path in the Easter egg hunt.



 ## Connect with Me
 - LinkedIn: https://www.linkedin.com/in/bahjath-nisa-023730265
 - Medium Blog: https://www.medium.com/@nisabahjath

  
