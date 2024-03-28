# Cross-Site Scripting (XSS) Testing Plan

## 1. Reconnaissance
- Use tools like Sublist3r, Amass, or Subfinder to enumerate subdomains for the in-scope wildcard domains
- Identify web applications, login forms, search functionality, and other potential entry points
- Create test accounts on the target applications if required

## 2. Manual Testing
- Manually explore the identified web applications and functionality
- Identify input fields, query parameters, and other user-controllable inputs
- Test each input field with basic XSS payloads:
  - `<script>alert(1)</script>`
  - `"><img src=x onerror=alert(1)>`
  - ...
- Check if the payload is reflected in the response without proper encoding or sanitization
- Test for stored XSS by submitting payloads in forms, comments, or user profiles and see if they persist
- Check for DOM-based XSS by analyzing client-side JavaScript and looking for unsafe dynamic code evaluation

## 3. Bypassing Filters
- If basic payloads are blocked or encoded, try advanced techniques to bypass filters:
  - Use different encoding schemes like HTML entities, Unicode, or hex encoding
  - Try alternate script tags:
    - `<scr<script>ipt>`
    - `<script/>`
    - `<script\s>`
  - Attempt to break out of existing HTML attributes using `">`, `'>`, or `-->`
  - Leverage event handlers like `onerror`, `onload`, `onfocus`, etc., instead of script tags
  - Experiment with different payloads and variations to see what bypasses the filters

## 4. Automation
- Use XSS scanning tools like XSStrike, OWASP ZAP, or Burp Suite to automate the testing process
- Configure the tools with the target URLs, login credentials (if required), and appropriate payloads
- Run the scans and review the results for any reported XSS vulnerabilities
- Manually verify the findings to eliminate false positives

## 5. Proof-of-Concept (PoC) Development
- For each identified XSS vulnerability, create a PoC that demonstrates the impact
- Craft payloads that execute JavaScript code, steal cookies, perform actions on behalf of the user, or deface the page
- Ensure the PoC clearly shows the vulnerability and its potential consequences
- Document the steps to reproduce the XSS vulnerability, including the affected URL, parameter, and payload

## 6. Reporting
- Compile your findings into a well-structured report
- Include a description of each XSS vulnerability, its location, and the steps to reproduce it
- Attach the PoC code or screenshots demonstrating the vulnerability
- Provide an assessment of the potential impact and severity of each finding
- Submit the report to InDrive's bug bounty platform, following their reporting guidelines



# SQL Injection Testing Plan for RESTful APIs

## 1. API Reconnaissance
- Identify the in-scope API endpoints:
 - profile-api.eu-east-1.indriverapp.com
 - truck-api.eu-east-1.indriverapp.com
 - ab-platform-api.eu-east-1.indriverapp.com
- Use tools like Postman, Insomnia, or Burp Suite to interact with the APIs
- Analyze the API documentation (if available) to understand the functionality and parameters
- Inspect the API requests and responses to infer the underlying data model and schema

## 2. Parameter Analysis
- For each API endpoint, identify the parameters that are passed in the requests
- Determine which parameters are used in database queries and are potential injection points
- Pay attention to parameters used in GET, POST, PUT, DELETE requests, as well as in URL paths and request bodies
- Note down any observed error messages, as they can provide insights into the backend database

## 3. SQL Injection Testing
- For each identified parameter, test for SQL injection vulnerabilities using various techniques:
 - UNION-based SQLi:
   - Attempt to combine the original query with a malicious query using the `UNION` keyword
   - Example payload: `' UNION SELECT username, password FROM users--`
 - Error-based SQLi:
   - Try to trigger database errors by injecting malformed SQL queries
   - Look for error messages that reveal sensitive information about the database structure
   - Example payload: `' OR 1=1--`
 - Time-based SQLi:
   - Attempt to introduce time delays in the database response by injecting conditional statements
   - Example payload: `' OR SLEEP(5)--`
 - Boolean-based blind SQLi:
   - Inject conditions that result in different responses based on the query outcome
   - Example payload: `' OR 1=1--` and `' OR 1=2--`
- Fuzz the parameters with a wide range of SQL injection payloads to maximize coverage
- Use tools like SQLMap or Burp Suite's SQLi scanner to automate the testing process

## 4. Exploitation and Impact Demonstration
- If an SQL injection vulnerability is found, attempt to exploit it further to demonstrate the impact
- Extract sensitive data from the database:
 - Retrieve user credentials, personal information, or other sensitive records
 - Example payload: `' UNION SELECT username, password FROM users--`
- Modify or delete records in the database:
 - Update or delete existing records to showcase the ability to manipulate data
 - Example payload: `'; UPDATE users SET password='hacked' WHERE user_id=1--`
- Explore the database schema and retrieve table and column names:
 - Use SQL information schema to map out the database structure
 - Example payload: `' UNION SELECT table_name, column_name FROM information_schema.columns--`

## 5. Reporting
- Document each identified SQL injection vulnerability in detail
- Include the affected API endpoint, parameter, and the SQL injection payload used
- Provide step-by-step instructions to reproduce the vulnerability
- Demonstrate the impact by showing the extracted sensitive data or modified records
- Assess the severity of the vulnerability based on the potential impact and ease of exploitation
- Submit the findings to InDrive's bug bounty platform, following their reporting guidelines




# Server-Side Request Forgery (SSRF) Testing Plan

## 1. Functionality Identification
- Analyze the in-scope domains and their functionality
- Look for features that accept URLs or allow loading of remote resources, such as:
 - File upload or download functionality
 - Image or document processors
 - URL shorteners or redirectors
 - PDF or thumbnail generators
 - XML or JSON parsers
 - Webhooks or callback URLs
- Identify any parameters or input fields that expect URLs or file paths

## 2. SSRF Testing
- For each identified functionality or parameter, test for SSRF vulnerabilities:
 - Provide malicious URLs that point to internal or external systems
 - Try different protocols, such as:
   - `http://` and `https://`
   - `file://`
   - `ftp://`
   - `gopher://`
   - `dict://`
 - Attempt to access internal IP addresses or hostnames:
   - Example payload: `http://localhost/`
   - Example payload: `http://127.0.0.1/`
   - Example payload: `http://internal.company.com/`
 - Test for access to metadata endpoints or cloud service URLs:
   - Example payload: `http://169.254.169.254/` (AWS metadata)
   - Example payload: `http://metadata.google.internal/` (GCP metadata)
 - Try to scan internal ports by providing URLs with different port numbers:
   - Example payload: `http://127.0.0.1:22/` (SSH port)
   - Example payload: `http://127.0.0.1:3306/` (MySQL port)
- Use URL encoding or other techniques to bypass any implemented SSRF filters:
 - Double URL encoding: `http%253A%252F%252F127.0.0.1%252F`
 - Using decimal IP representation: `http://2130706433/` (127.0.0.1)
 - Using hexadecimal IP representation: `http://0x7f000001/` (127.0.0.1)
- Leverage open-source tools like SSRFmap or Gopherus to automate SSRF testing

## 3. Impact Demonstration
- If an SSRF vulnerability is identified, attempt to demonstrate its impact:
 - Access sensitive data or files from internal systems
 - Perform port scanning to discover open ports and services
 - Attempt to access cloud metadata endpoints to retrieve sensitive information
 - Exploit trust relationships to access protected resources or escalate privileges
 - Trigger actions on internal services or cause denial-of-service conditions

## 4. Reporting
- Document each identified SSRF vulnerability in detail
- Specify the affected functionality, parameter, or input field
- Provide the malicious URLs or payloads used to trigger the SSRF
- Include step-by-step instructions to reproduce the vulnerability
- Demonstrate the impact by showcasing the accessed sensitive data or performed actions
- Assess the severity of the vulnerability based on the potential impact and ease of exploitation
- Submit the findings to InDrive's bug bounty platform, following their reporting guidelines
