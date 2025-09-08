# OWASP-Juice-Shop-Pentest
Penetration test report for OWASP Juice Shop covering common OWASP Top 10 vulnerabilities, POC screenshots, and remediation steps.

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

## Vulnerabilities

## DOM-Based XSS

### Explanation
The application is vulnerable to DOM Based XSS because it uses untrusted data from the URL and inserts it directly into the page using JavaScript, without sanitizing or validating the input.
This allows an attacker to inject malicious JavaScript code that runs in the victim’s browser.
###  Exploit Steps
- Access OWASP Juice shop: http://localhost:3000/ (if you have installed it locally)
- Go to the Search area and insert the below payload in search parameter
  <pre><iframe src="javascript:alert(`xss`)"></pre>
- After Inserting this payload as we press Enter our challenge gets completed and a alert box comes on screen with xss written on it.
### POC

<img width="980" height="394" alt="DOM-based XSS" src="https://github.com/user-attachments/assets/211bc256-fd5f-4fcf-baaf-93b2d1f76b0b" />

### Remediations
- Avoid using these when inserting untrusted input into the DOM:
`innerHTML, outerHTML, document.write, eval, setTimeout()` / `setInterval()` with strings, `Element.setAttribute()` with dynamic values like `src`, `href`, etc
Use safer alternatives: Use textContent or innerText to insert plain text safely
- Validate the input: Only allow expected input (like letters, numbers, etc.). Block special characters if not needed.
- Encode data before inserting it: Convert <, >, ', " to safe characters so they don’t run as code.
- Use a Content Security Policy (CSP)

## Missing Encoding
Vulnerability: Missing URL Encoding
Category: Improper Input Validation
Difficulty: ⭐ (Easy)

### Goal
Retrieve and view the photo of Bjoern's cat in "melee combat-mode" on the Photo Wall page.
### What is the vulnerability?
The app fails to properly encode special characters in image file names (like # or emoji). As a result:
The browser misinterprets part of the filename.
The image fails to load.
This is an example of "Missing Encoding" — where user input (like filenames or URLs) is not correctly encoded before use.
### Exploit Steps
- Go to the Photo Wall page.
- You’ll notice that one image is broken (it doesn’t load).
- Open DevTools (F12) → Go to the Network or Elements tab.
- Inspect the image tag (src="assets/public/images/uploads/ᓚᘏᗢ-#zatschi-#whoneedsfourlegs-1572600969477.jpg") of the broken image.
- The file name contains special characters, like #
- URLs cannot contain raw # characters, as # is used to indicate a fragment identifier in URLs.
- Use a URL encoder tool (https://meyerweb.com/eric/tools/dencoder/) to encode special characters.
- # become %23
- Replace the image tag manually with the encoded version (src="assets/public/images/uploads/ᓚᘏᗢ-%23zatschi-%23whoneedsfourlegs-1572600969477.jpg")
- Press Enter or paste it into the browser.
- The cat photo now loads successfully.
### POC
<img width="810" height="212" alt="missing encoding 2" src="https://github.com/user-attachments/assets/8587ae7b-6183-445a-af52-cfdc47c81f13" />


<img width="802" height="172" alt="missing encoding3" src="https://github.com/user-attachments/assets/123e86fe-96a3-4c6c-b710-9cc03893bb18" />


<img width="955" height="82" alt="missing encoding solved" src="https://github.com/user-attachments/assets/aa840e1f-9808-4ca2-a3bc-d5f8040b2bfc" />

### Root Cause
- The image URL was not encoded properly, so the browser couldn’t interpret the file path.
- This is a classic case of improper input handling: not encoding special characters before using them in URLs or file paths.
### Remediations
- Always encode special characters (#, &, ?, emoji, etc.) in URLs 
- Use `encodeURIComponent()` in JavaScript.

## 1.SQL Injection Login Bypass
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
<img width="1004" height="407" alt="SQLI" src="https://github.com/user-attachments/assets/299095fa-a1ef-49a0-8088-c1c4833b3802" />

<img width="1004" height="396" alt="SQLI2" src="https://github.com/user-attachments/assets/cf76954a-8d1d-4236-9310-d59b3c0c5320" />

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

  
