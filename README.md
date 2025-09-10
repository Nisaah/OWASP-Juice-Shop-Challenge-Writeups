# OWASP-Juice-Shop-solution-guide-difficulty-1 star
Welcome to my writeups for the [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/), one of the most modern and intentionally insecure web applications for security training.
This repository includes detailed walkthroughs for solving each challenge in Juice Shop, complete with explanations, screenshots, and best practices.

## Disclaimer
This repository is intended **only for educational and ethical purposes**. Do not use these techniques against systems you do not own or have explicit permission to test.

## About Me
I'm a cybersecurity enthusiast documenting my learning journey through hands-on labs and challenges. Feel free to connect or contribute!

## Tools Used
- Burpsuite
- Nmap
- SQLMap
- Firefox Developer tools

## Challenges (Difficulty: ⭐ Easy)

## 1. DOM-Based XSS
- Category: Improper Input Validation

### Explanation
The application is vulnerable to DOM Based XSS because it uses untrusted data from the URL and inserts it directly into the page using JavaScript, without sanitizing or validating the input.
This allows an attacker to inject malicious JavaScript code that runs in the victim’s browser.

###  Solution Steps
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

## 2. Missing Encoding
- Category: Improper Input Validation

### Goal
Retrieve and view the photo of Bjoern's cat in "melee combat-mode" on the Photo Wall page.

### What is the vulnerability?
The app fails to properly encode special characters in image file names (like # or emoji). As a result:
The browser misinterprets part of the filename.
The image fails to load.
This is an example of "Missing Encoding" — where user input (like filenames or URLs) is not correctly encoded before use.

### Solution Steps
- Go to the Photo Wall page.
- You’ll notice that one image is broken (it doesn’t load).
- Open DevTools (F12) → Go to the Network or Elements tab.
- Inspect the image tag (src="assets/public/images/uploads/ᓚᘏᗢ-#zatschi-#whoneedsfourlegs-1572600969477.jpg") of the broken image.
- The file name contains special characters, like #
- URLs cannot contain raw # characters, as # is used to indicate a fragment identifier in URLs.
- Use a URL encoder tool (https://meyerweb.com/eric/tools/dencoder/) to encode special characters.
- '#' become %23
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

## 3. Exposed Metrics
- Category: Sensitive Data Exposure

### Goal
Find the endpoint (URL) that exposes internal usage metrics to monitoring tools.

### Usage Metrics
- Metrics are data points about how the system is running and how it’s performing.
- Examples: CPU usage, number of users, page load times, error rates.
- These are usually collected by tools like Prometheus to monitor app health.
- They help developers, system admins, and cybersecurity teams monitor the health, performance, and behavior of the system.
  
### Prometheus
- Prometheus is an open-source monitoring system
- It collects metrics from apps via a URL endpoint (usually /metrics).
  
### Solution Steps
- Read the Hint: The hint tells you the monitoring system is Prometheus
- Check Prometheus Documentation: The docs show the default metrics endpoint is, /metrics
- Try That in Juice Shop: If your Juice Shop is running at
<pre>http://localhost:3000</pre>
- Then go to
<pre>http://localhost:3000/metrics</pre>
- That page will show internal app metrics — and boom! The challenge is solved.

### POCs

<img width="728" height="298" alt="exposed metrics" src="https://github.com/user-attachments/assets/aed97dc8-3ee8-4180-a449-483c8d1a2fc8" />

<img width="930" height="80" alt="exposed metrics--" src="https://github.com/user-attachments/assets/2166b558-ad88-4f6e-b591-ffc4ccc694ca" />

### Remediations
- Change default, easy-to-guess URL (/metrics) to a custom path (e.g., /hidden-stats-XY12)
- Add authentication: Require login, token, or API key to access /metrics
- Restrict access to /metrics: Only allow trusted IP addresses or internal users to see the endpoint
- Disable metrics in production: If metrics aren't needed in production, turn them off completely

## 4. Outdated Whitelist
- Category: Unvalidated Redirects (also called Open Redirects)

### Goal
Find a redirect link in the Juice Shop app that points to an old cryptocurrency address that’s no longer being promoted.

### Solution Steps:
- Open Developer Tools
- Find the Main JavaScript File from debugger tab (```main.js```,```main-es2018.js```)
- Use Pretty Print
      Minified files are hard to read — everything is on one long line.
      So, Click the {} icon at the bottom of the code window in the developer tools.
      This is called “Pretty Print”.
      It reformats the file so the code is nicely indented and readable.
      Alternatively, you can copy the code and paste it into an online JavaScript beautifier like: https://beautifier.io/
- Search the Beautified JavaScript: Use the search feature (usually Ctrl + F or Cmd + F) and look for keywords like, redirect, bitcoin, blockchain, address, wallet
- I found the url When I search blockchain, url: './redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm'
- Test the Link in the Browser, http://localhost:3000/redirect?to=https://blockchain.info/address/IAbKfgvw9psQ41NbLi8kufDQTezwG8DRZm

### POC

<img width="1004" height="284" alt="Outdated-Whitelist" src="https://github.com/user-attachments/assets/26b05a4d-8847-4842-9781-4a268c9d5327" />

<img width="1004" height="348" alt="Outdated-Whitelist2" src="https://github.com/user-attachments/assets/9622742e-8896-42e7-8dae-dbcb743ee25f" />


### Remediations
- Use a Whitelist of Allowed URLs: Only allow redirects to trusted, pre-approved domains. Example: allow only your own domain or known internal links.
- Block All External Redirects: If not absolutely necessary, do not allow redirection to external URLs at all.
- Validate the to= Parameter Carefully: Ensure the to value is a safe, expected format, not just any URL the user provides.
- Do Not Rely on Client-Side Redirects Only: Check and enforce redirect validation on the server side, not just in JavaScript.
- Use Relative Paths Instead of Full URLs: Instead of allowing full URLs like https://example.com, only allow internal paths like /dashboard.

## 5. Repetitive Registration

- Category: Improper Input Validation
- Hint: DRY = Don’t Repeat Yourself

### Goal
Follow the DRY principle while registering a user.

### DRY Principle (Don’t Repeat Yourself)
- It is a principle of software development to avoid repetition of code or information. 
- Instead of repetition of code, Write it once and reuse it wherever needed.
- **Example in Code (Not DRY)**:
      <pre>print("Welcome, Alice!")
      print("Welcome, Bob!")
      print("Welcome, Charlie!")</pre>

- **DRY Version**:
      <pre>def welcome(name):
        print(f"Welcome, {name}!")
      welcome("Alice")
      welcome("Bob")
      welcome("Charlie")</pre>

### Solution Steps
- Go to the registration page in Juice Shop.
- Fill in all fields normally:
    Email: test2@test.com
    Password: test1234
    Repeat Password: test1234
    then change thw Password field to test1234test, keep the Repeat Password: test1234
    No client-side validation error is shown in Repeat Password field
    Security Question: Any
    Answer: Any
- Submit the registration form
- Th Challenge is solved

### POCs

<img width="266" height="341" alt="DRY " src="https://github.com/user-attachments/assets/7516a256-6ac0-469a-a69b-9768d095bc40" />

<img width="940" height="81" alt="DRY1" src="https://github.com/user-attachments/assets/455d35e9-32bc-4b7e-90b5-4bdea082fae4" />

### Remediations
- Follow DRY Principle: Avoid the use of duplication of code
- Add Proper Server-Side Validation: The server should check that the password confirmation matches the password
- Improve Client-Side Validation: Ensure the UI checks and reject submission when the passwords don’t match


 ## Connect with Me
 - LinkedIn: https://www.linkedin.com/in/bahjath-nisa-023730265
 - Medium Blog: https://www.medium.com/@nisabahjath

  
