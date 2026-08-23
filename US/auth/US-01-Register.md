# User Story Specification: [US-01] [Academic Email Registration & Rate Limiting]

## 1. User Story Statement
* **As a** A university student 
* **I want to** register in the platform
* **So that** I want to competitive leagues, gain practical technical experience, So I can participate in, and improve my resume.
---

## 2. Story Metadata
| Attribute | Details / Value |
| :--- | :--- |
| **Story ID** | `US-01` |
| **Feature Area / Epic** | Authentication & Verification |
| **Priority** | Must Have (P1)  |
| **Story Points** | 5 |
| **SRS Requirements** | FR-AUTHENT-01, FR-AUTHENT-02, FR-AUTHENT-05, NFR-SEC-03 |
| **Sequence Diagram Reference** |  [`sign-up.mmd`](../diagrams/sign-up.mmd) |

---

## 3. Pre-Conditions & Post-Conditions

### Pre-Conditions
- [ ] The user is an unauthenticated visitor submitting a signup request.
- [ ] System Rate Limiter Gate and SMTP Email Gateway are active and reachable.
- [ ] Academic domain validation filter (`*.edu.*`) is configured.

### Post-Conditions
- [ ] Pending user record is persisted in PostgreSQL with a securely hashed password (BCrypt).
- [ ] A 6-digit OTP code with an expiration timestamp is stored and dispatched to the user's email via SMTP.
- [ ] The Rate Limiter counter increments for the client's IP address and email.

---

## 4. Acceptance Criteria (Gherkin Format)

### Scenario 1: Successful Account Creation & OTP Dispatch (Happy Path)
* **Given** an unregistered student using a university email (student@university.edu.*)
* **And** the request frequency is within rate limits (< 3 requests per 30-minute window)
* **When** the client submits a POST /api/v1/auth/signup request with required user details
* **Then** the system hashes the password, creates a pending user record in PostgreSQL, generates a 6-digit OTP, dispatches an email via SMTP, and returns HTTP `201 Created`.

### Scenario 2: Existing Account Conflict
* **Given** a registration request submitted with an academic email that already exists in PostgreSQL
* **When** the system checks the email existance during the account setup
* **Then** the process halts and return HTTP `409 Conflict` with the message "Account already exists"

### Scenario 3: Non-Academic Email Rejection
* **Given** a user attempts to register with a standard public email (e.g `user@gmail.com`)
* **When** the Domain Validator checks the email suffix
* **Then** the system rejects the payload and returns HTTP `400 Bad Request` with the message "Registration is restricted to official academic email domains (*.edu)".

### Scenario 4: Rate Limit Exceeded
* **Given** a client IP or email address that has attempted 3 signup requests within a 30-minute window.
* **When** the client submits a 4th signup request within that window  
* **Then** the Rate Limiter Gate intercepts the request and returns HTTP 429 Too Many Requests with the message "Too many signup attempts. Please try again later."
---

## 5. Technical & API Specifications

### Primary API Endpoint
* **HTTP Method:** `POST`
* **Endpoint Path:** `/api/v1/auth/signup`
* **Authentication:** `Public / Unauthenticated`
* **Authorized Roles:** `Guest / Anonymous Client`

### Header Specifications
```http
Content-Type: application/json
X-Forwarded-For: <CLIENT_IP>
```
### Request Body Schema Example
```json
{
  "email": "saja.ghoul@university.edu",
  "password": "SecurePassword123!",
  "full_name": "Saja Al-Ghoul",
  "university_name": "Engineering University"
}
```
### Success Response Example (HTTP 201 Created)
```json
{
  "status": "success",
  "code": 201,
  "message": "Pending account created successfully. Verification OTP dispatched.",
  "data": {
    "user_id": "a1b2c3d4-e5f6-7890-1234-56789abcdef0",
    "email": "firstname.lastname@university.edu.ps",
    "account_status": "PENDING_VERIFICATION",
    "otp_expires_in_seconds": 300
  }
}
```
### Error Response Example (HTTP 429 Too Many Requests)
```json
{
  "status": "error",
  "code": 429,
  "message": "Rate limit exceeded: Maximum 3 OTP/signup requests allowed within 30 minutes."
}
```

