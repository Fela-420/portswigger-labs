# Lab Documentation: Password Reset Broken Logic

## Lab

**Name:** Password Reset Broken Logic
**Difficulty:** Apprentice

## Objective

Exploit a flaw in the password reset process to reset Carlos's password without possessing a valid password reset token, then log in as Carlos and access the **My Account** page.

---

# Steps Performed

### 1. Request a Password Reset

* Logged in as:

  * **Username:** wiener
  * **Password:** peter
* Clicked **Forgot your password?**
* Entered the username **wiener**.
* Opened the password reset email using the **Email client**.

### 2. Observe the Reset Process

* Clicked the password reset link.
* Set a new password.
* Intercepted the requests using Burp Suite.
* Noticed that the reset token was included as a URL query parameter.

Example:

```http
GET /forgot-password?temp-forgot-password-token=TOKEN
```

### 3. Analyze the Password Reset Request

The password reset form submitted a POST request containing:

* Username
* New password
* Password confirmation
* Password reset token

The username was sent as a hidden form parameter.

### 4. Test Token Validation

Sent the password reset request to **Burp Repeater**.

Removed the value of the parameter:

```text
temp-forgot-password-token
```

from:

* The URL
* The request body

The password reset still succeeded.

**Finding:** The application was not validating the password reset token.

### 5. Exploit the Vulnerability

Requested another password reset for **wiener**.

Sent the POST request to Burp Repeater.

Modified:

* `username=wiener` → `username=carlos`
* Chose a new password.
* Left the reset token empty.

Sent the modified request.

The application accepted the request and reset Carlos's password.

### 6. Verify the Attack

Logged in using:

* **Username:** carlos
* **Password:** (new password chosen)

Successfully accessed **My Account**, solving the lab.

---

# Key Findings

* The application failed to validate the password reset token.
* The server trusted the username supplied by the client.
* Any user's password could be reset by modifying the username parameter.

# Vulnerability

**Broken Password Reset Logic**

The application accepted password reset requests without verifying that:

* The reset token was valid.
* The reset token belonged to the specified user.

# Impact

An attacker can:

* Reset any user's password.
* Take over arbitrary accounts.
* Gain unauthorized access without knowing the victim's credentials.

# Root Cause

The server relied on client-supplied parameters instead of validating the password reset token against the intended user.

# Remediation

* Always validate password reset tokens on the server.
* Bind each reset token to a specific user account.
* Reject missing, expired, or invalid tokens.
* Do not trust client-supplied usernames during password reset.
* Invalidate reset tokens immediately after successful use.
