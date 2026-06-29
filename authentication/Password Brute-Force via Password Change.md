# Password Brute-Force via Password Change – PortSwigger Lab

## Objective

Exploit an information disclosure vulnerability in the password change functionality to discover another user's password and access their account.

## Summary

The password change endpoint returned different error messages depending on whether the supplied current password was correct. This allowed password enumeration by brute-forcing the `current-password` parameter.

## Method

* Logged in using the provided credentials (`wiener:peter`).
* Captured the password change request in Burp Suite.
* Sent the request to Intruder.
* Changed the `username` parameter from `wiener` to `carlos`.
* Added a payload position to the `current-password` parameter.
* Loaded the provided candidate password list.
* Configured **Grep Match** to detect the response **"New passwords do not match"**.
* Ran the Intruder attack and identified the correct password based on the unique response.
* Logged in as `carlos` to complete the lab.

## Key Learning

Authentication-related features, such as password change forms, can unintentionally leak information through different error messages. Applications should use generic responses and implement brute-force protection across all authentication endpoints, not just the login page.

**Skills Practiced:** Burp Suite Intruder, password enumeration, request modification, brute-force testing, and authentication logic analysis.
