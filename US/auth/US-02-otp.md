# User Story Specification: [US-02] OTP Account Verification

## 1. User Story Statement
* **As an** Unverified Student
* **I want to** submit the 6-digit One-Time Password (OTP) sent to my academic email address
* **So that** I can verify my email ownership, activate my account, and receive a JWT authentication token to access the platform.

---

## 2. Story Metadata
| Attribute | Details / Value |
| :--- | :--- |
| **Story ID** | `US-02` |
| **Feature Area / Epic** | Authentication & Verification |
| **Priority** | Must Have (P1) |
| **Story Points** | 3 |
| **SRS Requirements** | FR-AUTHENT-02, FR-AUTHENT-04 |
| **Sequence Diagram Reference** | [`sign-up.mmd`](../diagrams/sign-up.mmd) |

---

## 3. Pre-Conditions & Post-Conditions

### Pre-Conditions
- [ ] A pending user account exists in PostgreSQL created via the signup endpoint.
- [ ] An active, unexpired 6-digit OTP code is stored in the database associated with the user's email.

### Post-Conditions
- [ ] User status in PostgreSQL updates from `PENDING` to `ACTIVE` (`is_verified = true`).
- [ ] The submitted OTP is invalidated/cleared to prevent reuse.
- [ ] A signed JSON Web Token (JWT) is generated and returned to the client.

---

## 4. Acceptance Criteria (Gherkin Format)

### Scenario 1: Successful OTP Verification & JWT Issuance (Happy Path)
* **Given** a pending student account with an unexpired OTP code (`123456`) in the database
* **When** the student submits a `POST /api/v1/auth/verify-otp` request containing their email and code `123456`
* **Then** the database validates the match, activates the user, clears the OTP record, and returns HTTP `200 OK` with a valid authorization JWT.

### Scenario 2: Invalid or Expired OTP Submission
* **Given** a pending student account and a submitted OTP code that is incorrect or older than its expiration threshold
* **When** the student sends the verification request
* **Then** the system rejects the attempt and returns HTTP `400 Bad Request` with the message `"Invalid or expired OTP"`.

### Scenario 3: Verification Attempt on Already Activated Account
* **Given** a student account that is already marked as verified and active
* **When** a verification request is received for that email
* **Then** the system rejects the request with HTTP `400 Bad Request` stating `"Account is already activated"`.

---

## 5. Technical & API Specifications

### Primary API Endpoint
* **HTTP Method:** `POST`
* **Endpoint Path:** `/api/v1/auth/verify-otp`
* **Authentication:** Public / Anonymous
* **Authorized Roles:** Unverified User / Guest

### Header Specifications
```http
Content-Type: application/json
```
### Request Body
```json
{
  "email": "saja.ghoul@university.edu",
  "otp_code": "482910"
}
```
### Sucess Response Example(HTTP 200 OK)
```json
{
  "status": "success",
  "code": 200,
  "message": "Account successfully activated.",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 86400,
    "user": {
      "user_id": "a1b2c3d4-e5f6-7890-1234-56789abcdef0",
      "email": "saja.ghoul@university.edu",
      "role": "STUDENT",
      "is_verified": true
    }
  }
}
```
### Error Response Example (HTTP 400 Bad Request)
```json
{
  "status": "error",
  "code": 400,
  "message": "Invalid or expired OTP code."
}
```