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
## 1. Accessing Hidden Files via robots.txt and Null Byte Injection in Juice Shop
### Explanation
robots.txt Discovery
While exploring the application, accessed the /robots.txt file: https://juice-shop.herokuapp.com/robots.txt
and saw:
<code>User-agent: *
Disallow: /ftp</code>

```Disallow: /ftp```
 ## Connect with Me
 - LinkedIn: https://www.linkedin.com/in/bahjath-nisa-023730265
 - Medium Blog: https://www.medium.com/@nisabahjath

  
