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
