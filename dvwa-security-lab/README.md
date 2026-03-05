# DVWA Security Lab Report

## SQL Injection

### Security Level: 
Low 🟡

### Payload:
```
1' OR '1'='1
```
##### Payload Source: OWASP Web Security Testing Guide

### Result:
The application returned multiple user records from the database instead of a single record. The output displayed several users including **admin, Gordon Brown, Hack Me, Pablo Picasso, and Bob Smith**.

### Screenshot:
![SQL Injection Low](screenshots/sql-injection-low.png)

### Explanation of why it worked:
The application builds an SQL query using unsanitized user input. A typical query pattern is:

    SELECT first_name, last_name
    FROM users
    WHERE user_id = '$id';

When the payload is supplied, the logic becomes:

    WHERE user_id = '1' OR '1'='1'

Because `'1'='1'` is always true, the database returns **all rows**, so multiple users are shown.

### Explanation of why it failed at higher level:
At higher security levels (Medium/High), DVWA applies stricter input handling e.g., validation and safer query parameters. These controls prevent special characters like `'` from changing the SQL syntax, so the injected condition cannot modify the query logic and the attack fails.

### Security Level: 
Medium 🟢

### Payload:
```
1' OR '1'='1
```
##### Payload Source: OWASP Web Security Testing Guide

### Result:
The payload could not be injected because the application replaced the text input field with a dropdown menu containing predefined user IDs. When selecting `User ID = 1` and submitting, the application returned only the corresponding user record:

**First name:** admin  
**Surname:** admin  

### Screenshot:
![SQL Injection Medium](screenshots/sql-injection-medium.png)

### Explanation of why it worked:
At the Medium security level, the application restricts user input by replacing the text field with a dropdown menu. This prevents users from entering arbitrary input containing special characters such as `'`, `OR`, or other SQL syntax.

Because the attacker cannot modify the input value, the SQL query cannot be manipulated.

### Explanation of why it failed at higher level:
Since the application restricts input to predefined values through the dropdown menu, it prevents malicious SQL payloads from being submitted. As a result, the attacker cannot alter the structure of the SQL query, and the injection attempt fails.

### Security Level:
High 🔴

### Payload:
```
1' OR '1'='1
```

##### Payload Source: OWASP Web Security Testing Guide

### Result:
The payload was entered in the SQL Injection session input window. However, unlike the Low security level, the application did not return multiple user records. Only a single record (admin) was shown.

### Screenshot:
![SQL Injection High](screenshots/sql-injection-high.png)

### Explanation of why it worked:
The payload tries to change the logic of the SQL query by adding the condition `'1'='1'`, which is always true. In systems that do not properly validate user input, this condition can cause the database to return all rows from the table.

### Explanation of why it failed at higher level:
At the High security level, DVWA applies stronger protections to user input. The application processes the input more carefully before using it in the SQL query, which prevents the injected condition from changing the query logic. Because of this, the attack does not return all users and only a normal result is shown.

## Command Injection

### Security Level: 
Low 🟡

### Payload:
```
127.0.0.1; id
```

##### Payload Source:  
Hackviser – Command Injection Testing Guide  
https://hackviser.com/tactics/pentesting/web/command-injection

### Result:
The application first executed the normal `ping` command and then also executed the `id` command. The output displayed:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This shows that the command was executed on the server.

### Screenshot:
![Command Injection Low](screenshots/command-injection-low.png)

### Explanation of why it worked:
The application directly uses the user input in a system command. The semicolon (`;`) allows another command to run after the first one. Because there is no input validation, the server runs both `ping` and `id`.

### Explanation of why it failed at higher level:
At higher security levels, the application filters or blocks special characters like `;`. Because of this, additional commands cannot be executed.

### Security Level: 
Medium 🟢

### Payload:
```
127.0.0.1 | id
```

##### Payload Source:  
Hackviser – Command Injection Testing Guide  
https://hackviser.com/tactics/pentesting/web/command-injection

### Result:
After submitting the payload, the application executed the injected `id` command and displayed:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This shows that the command was executed by the web server process, confirming that command injection is still possible at the Medium security level.

### Screenshot:
![Command Injection Medium](screenshots/command-injection-medium-recent.png)

### Explanation of why it worked:
At the Medium level, the application blocks some characters such as `;`, but other command operators like `|` are still allowed. By using the pipe operator, the injected `id` command was executed by the system.

### Explanation of why it failed at higher level:
At the High security level, stricter input validation is applied and more command operators are filtered, preventing additional commands from being executed.

### Security Level: 
High 🔴

### Payload:
```
127.0.0.1; id
```

##### Payload Source:  
Hackviser – Command Injection Testing Guide  
https://hackviser.com/tactics/pentesting/web/command-injection

### Result:
After submitting the payload, the application only executed the normal `ping` command. The output of the `id` command was not displayed, indicating that the injected command was not executed.

### Screenshot:
![Command Injection High](screenshots/command-injection-high.png)

### Explanation of why it failed:
At the High security level, the application performs stricter input validation and filtering. Special characters such as `;` used to chain commands are blocked or ignored. As a result, only the intended `ping` command is executed.

### Explanation of mitigation:
The application restricts the input and filters dangerous characters before executing the system command. This prevents attackers from injecting additional commands into the system call.

## Cross-Site Request Forgery (CSRF)

### Security Level:
Low 🟡

### Payload:
```html
<html>
<body>

<form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
  <input type="hidden" name="password_new" value="hacked123">
  <input type="hidden" name="password_conf" value="hacked123">
  <input type="hidden" name="Change" value="Change">
</form>

<script>
document.forms[0].submit();
</script>

</body>
</html>
```

##### Payload Source:
Hackviser – CSRF Testing Guide  
https://hackviser.com/tactics/pentesting/web/csrf

### Result:
The malicious HTML page automatically submitted the request and the password was changed without the user pressing the **Change** button.

### Screenshot:
![CSRF Low](screenshots/csrf-low.png)

### Explanation of why it worked:
At the Low security level, DVWA does not verify the origin of the request. Since the user is already authenticated, the browser automatically sends the session cookie with the request, allowing the password change to occur.

### Explanation of why it failed at higher level:
At higher security levels, DVWA introduces protections such as checking the HTTP Referer header and using CSRF tokens. These mechanisms ensure that requests originate from legitimate pages, preventing the malicious request from being processed.

### Security Level:
Medium 🟢

### Payload:
```html
<html>
<body>

<form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
  <input type="hidden" name="password_new" value="hacked123">
  <input type="hidden" name="password_conf" value="hacked123">
  <input type="hidden" name="Change" value="Change">
</form>

<script>
document.forms[0].submit();
</script>

</body>
</html>
```

##### Payload Source:
Hackviser – CSRF Testing Guide  
https://hackviser.com/tactics/pentesting/web/csrf

### Result:
The attack failed and the application displayed the message:  
`That request didn't look correct.`

### Screenshot:
![CSRF Medium](screenshots/csrf-medium.png)

### Explanation of why it worked:
At the Low security level, the application does not verify the origin of the request. Since the victim is already authenticated, the browser automatically sends the session cookie with the request, allowing the password change to occur.

### Explanation of why it failed at higher level:
At the Medium security level, DVWA checks the HTTP Referer header to ensure that the request originates from the DVWA application. Since the malicious request came from an external HTML page, the server rejected the request.

### Security Level:
High 🔴

### Payload:
```html
<html>
<body>

<form action="http://localhost:8080/vulnerabilities/csrf/" method="GET">
  <input type="hidden" name="password_new" value="hacked123">
  <input type="hidden" name="password_conf" value="hacked123">
  <input type="hidden" name="Change" value="Change">
</form>

<script>
document.forms[0].submit();
</script>

</body>
</html>
```

##### Payload Source:
Hackviser – CSRF Testing Guide  
https://hackviser.com/tactics/pentesting/web/csrf

### Result:
The attack failed and the application displayed the message:  
`CSRF token is incorrect`.

### Screenshot:
![CSRF High](screenshots/csrf-high.png)

### Explanation of why it worked:
At the Low security level, the application does not verify the origin of the request, allowing the malicious request to change the password.

### Explanation of why it failed at higher level:
At the High security level, DVWA requires a valid CSRF token to be included in the request. Since the malicious HTML page does not contain the correct token generated by the application, the request is rejected.

## File Inclusion

### Security Level:
Low 🟡

### Payload:
```
../../../../../etc/passwd
```

##### Payload Source:  
Hackviser – File Inclusion Testing Guide  
https://hackviser.com/tactics/pentesting/web/local-file-inclusion

### Result:
The application displayed the contents of the system file `/etc/passwd`, confirming that Local File Inclusion was possible.

### Screenshot:
![File Inclusion Low](screenshots/file-inclusion-low.png)

### Explanation of why it worked:
At the Low security level, the application directly includes the value provided in the `page` parameter without validating or sanitizing it. This allows an attacker to perform directory traversal and access sensitive files on the server.

### Explanation of why it failed at higher level:
At higher security levels, DVWA restricts file inclusion to specific allowed files and sanitizes user input, preventing directory traversal attacks.

### Security Level:
Medium 🟢

### Payload:
```
..//..//..//..//etc/passwd
```

##### Payload Source:
Hackviser – File Inclusion Testing Guide  
https://hackviser.com/tactics/pentesting/web/local-file-inclusion

### Result:
The application displayed the contents of `/etc/passwd`, confirming that the directory traversal filter was bypassed.

### Screenshot:
![File Inclusion Medium](screenshots/file-inclusion-medium.png)

### Explanation of why it worked:
At the Medium security level, the application attempts to block directory traversal by filtering the string `../`. However, the payload `..//` bypasses this filter while still resolving to a parent directory, allowing access to sensitive system files.

### Explanation of why it failed at higher level:
At higher security levels, DVWA restricts file inclusion to a predefined set of files and performs stricter input validation, preventing directory traversal attacks.

### Security Level:
High 🔴

### Payload:
```
file:///var/www/html/hackable/flags/fi.php
```

##### Payload Source:
Hackviser – File Inclusion Testing Guide  
https://hackviser.com/tactics/pentesting/web/local-file-inclusion

### Result:
The application displayed the contents of the file `/var/www/html/hackable/flags/fi.php`, confirming that file inclusion was still possible.

### Screenshot:
![File Inclusion High](screenshots/file-inclusion-high.png)

### Explanation of why it worked:
Although the High security level attempts to restrict file inclusion to specific filenames, the filter does not block PHP stream wrappers such as `file://`. This allows attackers to directly reference files on the server and bypass the intended restrictions.

### Explanation of why it failed at higher level:
Stronger implementations should strictly validate allowed file paths and disable dangerous wrappers such as `file://`, preventing arbitrary file inclusion.

## File Upload

### Security Level:
Low 🟡

### Payload:
```php
<?php system($_GET['cmd']); ?>
```

##### Payload Source:
Hackviser – File Upload Testing Guide  
https://hackviser.com/tactics/pentesting/web/file-upload

### Result:
The PHP file was uploaded successfully to the server and stored inside the `hackable/uploads` directory. By accessing the uploaded file through the browser, it was possible to execute system commands on the server.

### Screenshot:
![File Upload Low](screenshots/file-upload-low.png)

### Explanation of why it worked:
At the Low security level, the application does not validate the file type or extension of uploaded files. This allows attackers to upload malicious PHP scripts that can be executed on the server.

### Explanation of why it failed at higher level:
At higher security levels, the application performs validation checks such as restricting allowed file extensions or verifying MIME types, which prevents uploading executable scripts like PHP files.

### Security Level:
Medium 🟢

### Payload:
shell.php.jpg

```php
<?php system($_GET['cmd']); ?>
```

##### Payload Source:
Hackviser – File Upload Testing Guide  
https://hackviser.com/tactics/pentesting/web/file-upload

### Result:
The file was uploaded successfully using a double extension. Although the application attempted to restrict PHP uploads, the file bypassed the filter because the last extension was `.jpg`.

### Screenshot:
![File Upload Medium](screenshots/file-upload-medium.png)

### Explanation of why it worked:
The application only checks the file extension and allows files ending in `.jpg`. Using a double extension (`shell.php.jpg`) bypasses the validation while still containing executable PHP code.

### Explanation of why it failed at higher level:
At higher security levels, the application performs stricter validation such as checking MIME types or verifying the file content, which prevents uploading files containing executable code.

### Security Level:
High 🔴

### Payload:
shell.php.jpg

```php
GIF89a<?php system($_GET['cmd']); ?>
```

##### Payload Source:
Hackviser – File Upload Testing Guide  
https://hackviser.com/tactics/pentesting/web/file-upload

### Result:
The file was uploaded successfully using a disguised image payload. The file appeared to be an image but contained executable PHP code.

### Screenshot:
![File Upload High](screenshots/file-upload-high.png)

### Explanation of why it worked:
The payload includes valid GIF magic bytes which make the file appear as an image. The application allowed the upload because it passed basic image validation checks.

### Explanation of why it failed at higher level:
Secure implementations verify file contents more strictly and prevent execution of uploaded files, which blocks disguised malicious uploads.

## SQL Injection (Blind)

### Security Level:
Low 🟡

### Payload:
1 AND LENGTH(database())>1

##### Payload Source:
Hackviser – SQL Injection Testing Guide  
https://hackviser.com/tactics/pentesting/web/sql-injection

### Result:
The application returned the message **“User ID exists in the database.”**, indicating that the injected condition evaluated to true. This confirms that the input was executed as part of the SQL query.

### Screenshot:
![Blind SQL Injection Low](screenshots/sql-injection-blind-low.png)

### Explanation of why it worked:
The application directly inserted the user input into the SQL query without proper validation or parameterization. The payload added a boolean condition (`LENGTH(database())>1`) which was evaluated by the database. Since the DVWA database name (`dvwa`) has a length greater than 1, the condition returned true and the query executed successfully.

### Explanation of why it would fail at higher level:
Higher security levels implement input sanitization and prepared statements to prevent malicious SQL logic from being injected. These mechanisms restrict query manipulation and block unauthorized database operations, preventing blind SQL injection attacks.

### Security Level:
Medium 🟢

### Payload:
1 AND 1=2

### Result:
The response remained "User ID exists in the database".

### Screenshot:
![Blind SQL Injection Medium](screenshots/sql-injection-blind-medium.png)

### Explanation of why it worked
The application directly includes the user-supplied `id` parameter in the SQL query without proper validation. By injecting logical conditions such as `1 AND 1=1` or `1 AND 1=2`, the attacker can influence how the query is evaluated. The difference in the application's response allows the attacker to infer information about the database.

### Explanation of why it failed at higher level
At higher security levels, the application sanitizes the input and restricts the `id` parameter to numeric values. Because the input is cast to an integer before the query is executed, any injected SQL logic is removed. As a result, payloads such as `1 AND 1=2` are interpreted simply as `1`, preventing the SQL injection from affecting the query.

### Security Level:
High 🔴

### Payload:
1 AND SLEEP(5)

### Result:
The application response was delayed for several seconds, indicating that the injected SQL command was executed. This confirms the presence of a time-based blind SQL injection vulnerability.

### Screenshot:
![SQL Injection Blind High](screenshots/sql-injection-blind-high.png)

### Explanation of why it worked
The application uses a cookie value as part of the SQL query without proper validation. By injecting a payload such as `1 AND SLEEP(5)`, the attacker can force the database to pause execution. The delay in the application's response confirms that the injected SQL command was processed by the database.

### Explanation of why it failed at higher level
Secure implementations validate and sanitize cookie inputs before using them in database queries. Proper defenses such as parameterized queries, strict input validation, and prepared statements prevent malicious SQL commands from being executed, eliminating the possibility of time-based SQL injection attacks.

## Weak Session IDs

### Security Level:
Low 🟡

### Payload:
dvwaSession=5

### Result:
The session ID value increased sequentially each time the "Generate" button was pressed. Because the IDs follow a predictable pattern (1, 2, 3, ...), an attacker could guess valid session identifiers.

### Screenshot:
![Weak Session IDs Low](screenshots/weak-sesh-low-1.png)
![Weak Session IDs Low 2](screenshots/weak-sesh-low-2.png)

### Explanation of why it worked
The application generates session identifiers using a predictable incremental value rather than a cryptographically secure random generator. Because the IDs follow a simple sequence, attackers can easily predict valid session IDs and potentially hijack active sessions.

### Explanation of why it failed at higher level
Secure implementations generate session identifiers using strong cryptographic randomness and enforce proper session management. This prevents attackers from predicting or brute-forcing session identifiers.

### Security Level:
Medium 🟢

### Payload:
dvwaSession=1709812348

### Result:
The session ID values changed based on the current timestamp each time the "Generate" button was pressed. Because the session identifiers follow a predictable time-based pattern, an attacker can estimate or guess valid session IDs.

### Screenshot:
![Weak Session IDs Medium](screenshots/weak-sesh-medium.png)

### Explanation of why it worked
At the Medium security level, the application generates session identifiers using the current timestamp. Although this is slightly more complex than sequential numbers, timestamps are still predictable because they are based on the current time. An attacker can estimate nearby values and attempt to hijack active sessions.

### Explanation of why it failed at higher level
Secure implementations generate session identifiers using cryptographically secure random number generators. Random session tokens are extremely difficult to predict, preventing attackers from guessing valid session IDs.

### Security Level:
High 🔴

### Payload:
dvwaSession (no predictable value observed)

### Result:
The session ID remained the same even after clicking the "Generate" button multiple times. No sequential or time-based pattern was observed.

### Screenshot:
![Weak Session IDs High](screenshots/weak-sesh-high.png)

### Explanation of why it worked
At lower security levels, the application generated session identifiers using predictable patterns such as sequential numbers or timestamps. These patterns allowed attackers to guess valid session identifiers.

### Explanation of why it failed at higher level
At the High security level, the application relies on secure session management. The session ID is generated using a secure random mechanism and remains stable for the active session. Because the identifier is not predictable and does not change in a pattern, attackers cannot guess or manipulate session IDs.

## XSS (DOM)

### Security Level:
Low 🟡

### Payload:
?default=<script>alert(XSS)</script>

### Result:
A JavaScript alert box appeared when the page loaded, confirming that the injected script executed successfully.

### Screenshot:
![DOM XSS Low](screenshots/xss-dom-low.png)

### Explanation of why it worked
The application reads the value of the `default` parameter directly from the URL and inserts it into the DOM without sanitization. Because the input is not filtered or encoded, arbitrary JavaScript can be injected and executed in the browser.

### Explanation of why it failed at higher level
At higher security levels, the application validates or sanitizes the input before inserting it into the DOM. Potentially dangerous characters and script tags are filtered or encoded, preventing execution of injected JavaScript.

### Security Level:
Medium 🟢

### Payload:
?default=<img src=x onerror=alert(1)>

### Result:
A JavaScript alert box appeared when the page loaded, confirming that the injected payload executed successfully.

### Screenshot:
![DOM XSS Medium](screenshots/xss-dom-medium.png)

### Explanation of why it worked
At the medium security level, the application blocks simple `<script>` tags but still allows HTML elements with JavaScript event handlers. The injected `<img>` tag triggers the `onerror` event when the image fails to load, which executes the JavaScript payload.

### Explanation of why it failed at higher level
At higher security levels, the application applies stronger input validation and sanitization. Dangerous characters, tags, and event handlers are filtered or encoded before being inserted into the DOM, preventing execution of injected JavaScript.

### Security Level:
High 🔴

### Payload Attempted:
?default=<script>alert(1)</script>

### Result:
The payload did not execute. The application ignored the injected value and reverted to a valid predefined option (e.g., "English").

### Screenshot:
![DOM XSS High](screenshots/xss-dom-high.png)

### Explanation of why it worked at lower levels
At lower security levels, the application directly inserted the `default` parameter from the URL into the DOM without sanitization. This allowed attackers to inject malicious JavaScript code that executed in the browser.

### Explanation of why it failed at high level
At the high security level, the application restricts input to a predefined list of allowed language values. Any input outside these allowed values is ignored, preventing malicious scripts from being inserted into the DOM. This effectively mitigates the DOM-based XSS vulnerability.
