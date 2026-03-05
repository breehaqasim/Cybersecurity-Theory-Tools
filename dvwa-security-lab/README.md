# DVWA Security Lab Report

## SQL Injection

### Security Level: 
Low

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
Medium

### Attempted Payload:
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
