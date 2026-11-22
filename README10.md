# API Boilerplate Project (with Email Verification & Password Reset)

## Project Description

A production-ready backend RESTful API boilerplate built with Node.js, Express, TypeScript, and TypeORM. Designed for scalability and strict security, this project features centralized error management, strict request validation with Joi, structured logging, database migrations, and a comprehensive authentication suite—including stateless JSON Web Tokens (JWT), Role-Based Access Control (RBAC), **Email Verification**, and **Secure Password Resets**.

---

## Complete Authentication & Email Flows

### 1. User Registration & Email Verification Flow
1. **Registration:** The client submits registration details via `POST /api/users` (or `/api/auth/register`).
2. **Account Creation:** The server hashes the password with `bcrypt`, sets `isVerified: false`, and generates a cryptographically secure, 32-byte hexadecimal `verificationToken` with an expiry timestamp (24 hours).
3. **Email Dispatch:** An automated verification email containing a unique link (e.g., `https://frontend.com/verify-email?token=...`) is dispatched via Nodemailer.
4. **Verification:** The user clicks the link, sending a request to `GET /api/auth/verify-email`. The server validates the token, sets `isVerified: true`, and clears (`null`s) the token and expiry fields from the database.

### 2. Login & JWT Authentication Flow
1. **Credentials Submission:** The user submits their email and password to `POST /api/auth/login`.
2. **Verification Check:** The server verifies that the email exists, checks the password hash, and confirms that `isVerified === true`. Unverified accounts are blocked from accessing the application.
3. **JWT Issuance:** A signed JWT containing `userId` and `role` claims is returned.
4. **Request Authorization:** Subsequent HTTP requests carry the token in the `Authorization: Bearer <token>` header, verified on protected routes by the `requireAuth` middleware.

### 3. Password Reset Flow
1. **Reset Request:** The user submits their email to `POST /api/auth/forgot-password`.
2. **Anti-Enumeration Response:** Regardless of whether the email exists in the database, the server returns a generic `200 OK` response.
3. **Token Generation:** If the account exists, the server generates a high-entropy `resetPasswordToken` expiring in 15–60 minutes and emails a reset link to the user.
4. **Password Overwrite:** The user posts the token and their new password to `POST /api/auth/reset-password`. The server verifies the token's validity, hashes the new password, and immediately wipes `resetPasswordToken` and `resetPasswordExpiry` from the database.

---

## Testing Email Delivery with Ethereal

During local development, outgoing emails are captured by **Ethereal Email**—a fake SMTP service—preventing spam to real inboxes while allowing you to inspect rendered HTML and test links.

### How to View Sent Emails
1. **Run the Application:** Start the development server (`npm run dev`).
2. **Trigger an Email Event:** Register a new user or request a password reset.
3. **Inspect Terminal Logs:** Ethereal automatically logs the preview URL directly to your console:
   ```text
   ✉️ Verification email sent to user@example.com
   🔗 Ethereal Email Preview URL: https://ethereal.email/message/XxYz123456789
   ```
4. **Access the Preview:** Open the link in your browser to view the email, verify the layout, and click the contained verification or password reset token link.

### Environment Configuration (`.env`)
```env
# Ethereal SMTP (Development)
SMTP_HOST=smtp.ethereal.email
SMTP_PORT=587
SMTP_USER=x4xgqd6ugsf3lwr4@ethereal.email
SMTP_PASS=UfqgEZAWEj7fpwvWUx
EMAIL_FROM=noreply@yourapp.com
```

---

## API Endpoints Overview & Examples

### Public Endpoints

#### 1. Register User
* **Method & Path:** `POST /api/users`
* **Request Body:**
  ```json
  {
    "name": "Pokarii T. Villarin",
    "email": "pokarii@gmail.com",
    "password": "pokarii1234testing",
    "age": "20"
  }
  ```
* **Response (201 Created):**
  ```json
  {
    "message": "User registered successfully. Please check your email to verify your account."
  }
  ```

#### 2. Verify Email
* **Method & Path:** `GET /api/auth/verify-email?token=<verification_token>`
* **Response (200 OK):**
  ```json
  {
    "message": "Email verified successfully. You can now log in."
  }
  ```

#### 3. Login
* **Method & Path:** `POST /api/auth/login`
* **Request Body:**
  ```json
  {
    "email": "pokarii@gmail.com",
    "password": "pokarii1234testing"
  }
  ```
* **Response (200 OK):**
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 1,
      "name": "Pokarii T. Villarin",
      "email": "pokarii@gmail.com",
      "role": "user"
    }
  }
  ```

#### 4. Forgot Password
* **Method & Path:** `POST /api/auth/forgot-password`
* **Request Body:**
  ```json
  {
    "email": "pokarii@gmail.com"
  }
  ```
* **Response (200 OK):**
  ```json
  {
    "message": "If an account with that email exists, a password reset link has been sent."
  }
  ```

#### 5. Reset Password
* **Method & Path:** `POST /api/auth/reset-password`
* **Request Body:**
  ```json
  {
    "token": "4a7b9c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b",
    "newPassword": "MyNewSecurePassword456!"
  }
  ```
* **Response (200 OK):**
  ```json
  {
    "message": "Password has been reset successfully. You may now log in with your new password."
  }
  ```

---

### Protected Endpoints (Requires `Authorization: Bearer <token>`)

#### 6. Get Current Profile
* **Method & Path:** `GET /api/auth/me`
* **Headers:** `Authorization: Bearer <jwt_token>`
* **Response (200 OK):**
  ```json
  {
    "id": 1,
    "name": "Pokarii T. Villarin",
    "email": "pokarii@gmail.com",
    "role": "user",
    "isVerified": true
  }
  ```

---

### Restricted Endpoints (Requires Token + `admin` Role)

#### 7. Get All Users
* **Method & Path:** `GET /api/users`
* **Headers:** `Authorization: Bearer <admin_jwt_token>`
* **Response (200 OK):**
  ```json
  [
    {
      "id": 1,
      "name": "Pokarii T. Villarin",
      "email": "pokarii@gmail.com",
      "role": "user"
    }
  ]
  ```

#### 8. Delete User
* **Method & Path:** `DELETE /api/users/:id`
* **Headers:** `Authorization: Bearer <admin_jwt_token>`
* **Response (200 OK):**
  ```json
  {
    "message": "User deleted successfully."
  }
  ```

---

## 🛡️ Security Features Implemented

* **Cryptographic Token Generation:** Uses Node's built-in `crypto.randomBytes(32)` producing 256 bits of entropy ($2^{256}$ combinations) for verification and reset tokens.
* **Anti-Email Enumeration:** Implements uniform response messaging on authentication and password reset endpoints to block account discovery.
* **Single-Use Token Consumption:** Instantly clears tokens (`null`) upon successful verification or password reset to prevent token reuse and replay attacks.
* **Time-Bound Token Expirations:** Enforces strict window limits (24 hours for email verification; 15–60 minutes for password reset).
* **Strict Database Projection:** Sensitive fields like `password`, `verificationToken`, and `resetPasswordToken` are marked with `select: false` in TypeORM to prevent accidental leakage in queries.
* **Password Hashing:** Uses `bcrypt` with salt and cost factor iterations to store irreversible password hashes.
* **Stateless JWT Authorization:** Decouples session storage from server memory while protecting endpoints using custom authentication and admin middleware.

---

## Project Reflection

**1. Why is email verification important for security? What problems does it prevent?**
> Email verification is industry standard and this is one of the security verifications to make sure it's the owner whis doing any changes on the account. It prevents account hacking.

**2. Explain the "always return success" pattern in forgot password. Why is it a security best practice?**
> This is for hackers not to be able to track accounts that are not actually registered on the systems.

**3. What's the difference between a verification token and a reset token? Why do they have different expiration times?**
> A verification token proves that a user genuinely owns the email address they registered with, while a reset token grants temporary, high-privilege authority to overwrite an account's password without knowing the current one. They differ on expiration time because for verification (24 hours)for user experience as there are times delay oon getting it on one's email depending with email service provider while for reset (15-60 mins only) to minimize account hacking risks.

**4. Why do we use `crypto.randomBytes(32)` instead of `Math.random()` or `Date.now()` for token generation?**
> The `crypto.randomBytes(32)` generates cryptographically secure, unpredictable tokens with $2^{256}$ bits of entropy, whereas `Math.random()` and `Date.now()` are predictable and can be guessed or brute-forced by an attacker.

**5. What happens if a user never verifies their email? How does your system handle this?**
> If a user will not verify their email then they can't log into the system getting error for 'Please verify your email first'. That's to detect as well humans and not bot.

**6. Explain why tokens are cleared after use. What security risk does this prevent?**
> Tokens is for 1 time use for security and this will make sure whatever you set for a token, it's only for that user preventing still account takeover.

**7. What is email enumeration and how does your implementation prevent it?**
> Email enumeration is a security vulnerability where an application reveals whether a specific email address is registered in its database which typically through different error messages, HTTP status codes, or response times.Attackers exploit this by running automated scripts against endpoints like login or password reset to harvest a verified list of active user emails, which they then use for targeted phishing, credential stuffing, or brute-force attacks.

**8. If you had to implement two-factor authentication (2FA) on top of this system, how would you approach it?**
> 1. **Install 2FA Dependencies**
>    Install a TOTP library and QR code generator (e.g., `otplib` and `qrcode`).
>
> 2. **Extend the `User` Entity (`src/entities/User.ts`)**
>    Add the required 2FA columns using explicit column types for `tsx` compatibility, then generate and run a new database migration:
>    ```typescript
>    @Column({ type: 'boolean', default: false })
>    isTwoFactorEnabled!: boolean;
>
>    // Encrypted at rest using AES-256-GCM (not hashed, as the server needs it to verify TOTP codes)
>    @Column({ type: 'varchar', length: 255, nullable: true, select: false })
>    twoFactorSecret!: string | null;
>
>    // Array of bcrypt-hashed single-use recovery codes
>    @Column({ type: 'json', nullable: true, select: false })
>    twoFactorBackupCodes!: string[] | null;
>    ```
>
> 3. **Create the 2FA Enrollment Flow**
>    - **`POST /api/auth/2fa/setup` (Protected):** Generate a unique TOTP secret, encrypt and save it to `twoFactorSecret` (leaving `isTwoFactorEnabled` as `false`), and return a QR code data URL for the user to scan.
>    - **`POST /api/auth/2fa/verify-setup` (Protected):** Verify the user's first 6-digit code from their authenticator app, set `isTwoFactorEnabled = true`, generate 8–10 single-use backup codes, store their bcrypt hashes in `twoFactorBackupCodes`, and return the plaintext backup codes once for the user to save.
>
> 4. **Split the Login Flow into Two Steps**
>    - **Step 1 (`POST /api/auth/login`):** Verify the user's email and password with bcrypt. If `isTwoFactorEnabled` is `false`, return the normal access JWT. If `true`, return a short-lived (5-minute) restricted pending token instead:
>      ```typescript
>      const tempToken = jwt.sign(
>          { userId: user.id, tokenType: '2fa_pending' },
>          config.jwt.secret,
>          { expiresIn: '5m' }
>      );
>      return { requiresTwoFactor: true, tempToken };
>      ```
>    - **Step 2 (`POST /api/auth/2fa/authenticate`):** Accept the `tempToken` and the 6-digit TOTP code (or a backup code). Verify the `tempToken` (`tokenType === '2fa_pending'`), validate the 2FA code against the decrypted `twoFactorSecret` (or remove the matched backup code), and issue the full-access JWT.
>
> 5. **Harden the Auth Middleware (`src/middleware/auth.ts`)**
>    Reject pending 2FA tokens on standard protected routes so users cannot bypass the second factor:
>    ```typescript
>    if (decoded.tokenType !== 'access') {
>        return next(new UnauthorizedError('Full authentication required'));
>    }
>    ```
>
> 6. **Apply Essential Security Safeguards**
>    - **Encrypt secrets at rest:** Encrypt `twoFactorSecret` with AES-256-GCM using a dedicated key in `.env` before storing it in MySQL.
>    - **Enforce strict rate limiting:** Limit attempts on `/api/auth/2fa/authenticate` (e.g., 5 attempts per 5 minutes) to prevent brute-forcing 6-digit codes.
>    - **Protect deactivation & recovery:** Require the user's current password plus a valid 2FA or backup code to disable 2FA, and immediately delete each backup code after a single use.

**9. What are the trade-offs between using Ethereal (test service) vs. SendGrid (production service)?**
> Ethereal is a fake SMTP service built strictly for safe local development, whereas SendGrid is a production email delivery platform engineered to reliably route real emails into user inboxes at scale.

**10. How would you handle a situation where a user's email provider marks your emails as spam?**
> Can advise a user to add it on Safe Sender's list or contact ESP as it was added for whitelisting.
