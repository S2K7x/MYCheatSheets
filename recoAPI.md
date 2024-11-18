# 🛡️ API Reconnaissance and Security Testing Cheatsheet

## 📖 API Documentation and Discovery

### Discovering API Documentation
- **Sources for Documentation:**
  - Public repositories (e.g., GitHub, Bitbucket).
  - API directories (e.g., [RapidAPI](https://rapidapi.com)).
  - Developer portals or open Swagger specs (`/swagger.json` or `/api-docs` endpoints).
  - JavaScript files in web apps for hardcoded API references.
  - Webpage HTML comments and source code (`CTRL + U` to view source).

- **Automation Tools:**
  - [Swagger Inspector](https://swagger.io/tools/swagger-inspector/)
  - [Postman](https://www.postman.com/)
  - [Recon-ng](https://github.com/lanmaster53/recon-ng)

---

### Machine-Readable API Documentation
- Check for:
  - **Swagger/OpenAPI JSON or YAML files**: 
    ```bash
    curl -s https://target.com/api/swagger.json
    ```
  - **Postman Collections**: Often exported and found in public repositories.
  - **WSDL Files**: XML-based for SOAP APIs (`?wsdl` query string).

---

## 🚀 Identifying and Interacting with API Endpoints

### Identifying API Endpoints
- **Techniques:**
  - Use tools like `Burp Suite` or `ZAP Proxy` to intercept traffic.
  - Fuzz common paths using tools like `ffuf` or `dirsearch`:
    ```bash
    ffuf -u https://target.com/api/FUZZ -w wordlist.txt
    ```
  - Look for non-standard file extensions (`.json`, `.xml`, `.php`).
  - Use passive reconnaissance (Google Dorks):
    ```bash
    site:target.com "api" filetype:json
    ```

---

### Identifying Supported HTTP Methods
- Check for HTTP methods like `GET`, `POST`, `PUT`, `DELETE`, etc.:
  ```bash
  curl -X OPTIONS https://target.com/api/endpoint
  ```
- Tools:
  - [HTTPie](https://httpie.io/):
    ```bash
    http OPTIONS https://target.com/api/endpoint
    ```
  - Burp Suite's Repeater feature.

---

### Identifying Supported Content Types
- Test for accepted formats (e.g., `application/json`, `text/xml`):
  ```bash
  curl -X POST https://target.com/api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"test":"value"}'
  ```

---

### Finding Hidden Endpoints
- **Automation Tools:**
  - [Intruder in Burp Suite](https://portswigger.net/burp/documentation/desktop/tools/intruder) to enumerate hidden endpoints.
  - Tools like `ParamSpider`, `Arjun`, and `ffuf` for hidden parameters.

---

## 🔍 Exploiting Common API Vulnerabilities

### Mass Assignment Vulnerabilities
- **What It Is:** Overly permissive API design that allows user-supplied JSON or form data to modify sensitive fields.

- **Testing:**
  - Inspect API responses for fields and try adding them during requests:
    ```json
    {
      "username": "user1",
      "role": "admin"
    }
    ```
  - Tools: Burp Suite’s Repeater or custom scripts.

- **Mitigation:**
  - Implement allowlists for parameters.
  - Validate and sanitize inputs server-side.

---

### Server-Side Parameter Pollution (SSPP)
- **What It Is:** Manipulating query strings or paths to override parameters.

#### Testing Query String Pollution
- **Steps:**
  - Try truncating or injecting duplicate parameters:
    ```bash
    https://target.com/api/endpoint?user=admin&user=guest
    ```
  - Inject invalid or extra parameters:
    ```bash
    https://target.com/api/endpoint?unexpected=123
    ```

- **Payloads to Try:**
  - Truncated:
    ```
    https://target.com/api/endpoint?param%00
    ```
  - Duplicated parameters:
    ```
    ?id=1&id=2
    ```

#### Testing REST Path Pollution
- Look for vulnerable REST routes:
  ```bash
  https://target.com/api/users/1234/admin
  ```
- Use fuzzing tools (e.g., `ffuf`) to inject payloads into path segments.

---

## 🛡️ Preventing API Vulnerabilities

### General Best Practices
1. **Authentication and Authorization:**
   - Enforce strong authentication (OAuth 2.0, JWT tokens).
   - Use RBAC (Role-Based Access Control) for granular access control.
   
2. **Input Validation:**
   - Whitelist acceptable inputs.
   - Avoid implicit type conversion (e.g., `"123"` becoming `123`).

3. **Rate Limiting and Quotas:**
   - Implement rate-limiting to prevent abuse.
   - Tools: Cloudflare API Gateway, AWS API Gateway.

4. **Error Messages:**
   - Do not leak sensitive information in error responses.

5. **API Versioning:**
   - Clearly separate public and private API versions.

---

## 🔧 Tools for API Security Testing

| Tool          | Purpose                       | URL/Command                            |
|---------------|-------------------------------|----------------------------------------|
| Postman       | API testing and automation    | [Postman](https://www.postman.com/)    |
| Burp Suite    | Intercepting and fuzzing      | [Burp](https://portswigger.net/burp)   |
| ZAP Proxy     | Automated API scans           | [ZAP](https://www.zaproxy.org/)        |
| ffuf          | Fuzzing hidden endpoints      | `ffuf -u URL -w wordlist`              |
| Arjun         | Parameter discovery           | [Arjun](https://github.com/s0md3v/Arjun) |
| Swagger UI    | Visualize API endpoints       | [Swagger](https://swagger.io/tools/swagger-ui/) |
| Intruder      | Burp Suite's attack tool      | Burp Suite Intruder                   |

---

## 🛠️ Sample Attack Scenarios

### Example 1: Exploiting an Unprotected Endpoint
**Request:**
```http
POST /api/v1/admin/users HTTP/1.1
Host: target.com
Content-Type: application/json
Content-Length: 50

{
  "username": "testuser",
  "role": "admin"
}
```

**Response:**
```json
{
  "message": "User created successfully."
}
```

---

### Example 2: Finding Hidden Parameters
**Request:**
```http
GET /api/orders?debug=true HTTP/1.1
```
**Response:**
```json
{
  "order_details": [...],
  "debug_info": { "stack_trace": "...", "sql_query": "..." }
}
```
