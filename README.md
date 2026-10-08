# Crack the Gate (Header change) - CTF Write-up

## Overview

**Challenge:** Crack the Gate
**Platform:** CyLab
**Category:** Web Security / Authentication
**Difficulty:** Beginner
**Operating Systems:** Windows, Kali Linux
**Browsers:** Brave
**Tools Used:** cURL, Burp Suite, Browser Developer Tools, ROT13 Decoder

### Objective

The objective of this challenge was to gain access to the application through the login page. The email address was known, but the password was unknown.

The challenge required identifying an additional authentication mechanism hidden within the application's page source.

---

## 1. Initial Reconnaissance

I accessed the web application's login page and observed that it required an email address and password.

The email address was known, but the password was not.

Instead of attempting to guess or brute-force the password, I inspected the application for information that could reveal how the authentication mechanism worked.

---

## 2. Inspecting the Page Source

I inspected the source code of the login page and discovered an authentication-related header named:


The value associated with the header was encoded using **ROT13**.

### Screenshot
<img width="875" height="321" alt="Screenshot 2026-10-08 134036" src="https://github.com/user-attachments/assets/4a4a84f5-3812-4f67-bf7e-c7bd25906f0e" />

```text
![Page source showing encoded Access header](images/page-source.png)
```

---

## 3. Decoding the Header

I identified the encoding as ROT13 and decoded the value.

ROT13 is a substitution cipher that shifts each letter 13 positions through the alphabet. It is easily reversible and should not be considered a security mechanism.

The decoded value provided the value required for the `Access` header.

```text
Encoded:
ABGR: Wnpx - grzcbenel olcnff: hfr urnqre "K-Qri-Npprff: lrf"
Decoded:
NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes" 
```

---

# 4. Exploitation

I successfully solved the challenge using two different methods.

## Method 1: Using cURL on Kali Linux

For the first method, I used Kali Linux and the terminal.

I submitted the login request using `curl` and manually added the required `Access` header to the HTTP request.

The request was structured similar to:

```bash
curl -X POST "https://<challenge-link>/login" \
  -H "Access: <decoded-value>" \
  -d "email=ctf-player@cylabacademy.org"&password=123"
```

The important part of the request was the custom `Access` header:

After sending the request with the required header, the application accepted the authentication request and provided access to the protected functionality.

The flag was then retrieved.

### Screenshot

<img width="525" height="261" alt="Screenshot 2026-10-08 135407" src="https://github.com/user-attachments/assets/2cafdfa6-4303-4133-a53e-7a7d341c63e9" />

```text
![Successful authentication using cURL](images/curl-request.png)
```

---

# 5. Method 2: Using Burp Suite

I also solved the challenge using **Burp Suite**.

Instead of sending the request directly through the terminal, I intercepted the login request from the browser.

### Step 1: Intercept the Login Request

I configured Burp Suite to intercept the request generated when submitting the login form.

The intercepted request contained the login information submitted by the browser.

### Step 2: Add the Access Header

I modified the intercepted HTTP request by adding the decoded `Access` header.

The modified request contained:

For example:

```http
POST /login HTTP/1.1
Host: <challenge-link>
Content-Type: application/x-www-form-urlencoded
Access: <decoded-value>

email=<known-email>&password=<password>
```

I then forwarded the modified request.

### Step 3: Retrieve the Flag

The server accepted the modified request and granted access to the protected resource.

The flag was then captured.

### Burp suite Screenshot

<img width="1082" height="507" alt="Screenshot 2026-10-08 135841" src="https://github.com/user-attachments/assets/5ad634bb-2262-4709-9a1f-655f312b9b79" />

```text
![Burp Suite request with Access header](images/burp-request.png)
```

---

# 6. Attack Chain

The overall attack can be summarized as:

```text
Login Page
     ↓
Inspect Page Source
     ↓
Discover "Access" Header
     ↓
Identify ROT13 Encoding
     ↓
Decode Header Value
     ↓
Add Header to Login Request
     ↓
 ┌───────────────┬────────────────┐
 ↓               ↓
cURL           Burp Suite
 ↓               ↓
Modified       Intercepted
Request        Request
 └───────────────┴────────────────┘
                 ↓
        Authentication Bypass
                 ↓
             Access Granted
                 ↓
             Capture Flag
```

---

# 7. Vulnerability Analysis

The main weakness was the exposure of an authentication-related value in the client-side page source.

The value was only protected through **ROT13 encoding**, which provides no actual confidentiality.

The decoded value was then accepted through a custom `Access` HTTP header during authentication.

This created an authentication weakness because the information required to satisfy the application's access-control mechanism was available to the client.

### Key weaknesses

* Sensitive authentication information exposed in page source.
* ROT13 used as an obfuscation mechanism.
* Authentication dependent on a client-supplied custom header.
* Insufficient server-side protection of the authentication mechanism.

---

# 8. Security Lessons

### ROT13 is not encryption

ROT13 is an encoding technique and can be reversed immediately. It should never be used to protect passwords, API keys, authentication tokens, or other sensitive information.

### Client-side information should not contain authentication secrets

Anything delivered to the browser can potentially be inspected by the user.

### HTTP headers can be manipulated

Custom HTTP headers are controlled by the client and should not automatically be trusted as proof of authorization.

### Burp Suite is useful for understanding web applications

Intercepting requests makes it possible to inspect and modify HTTP requests before they reach the server. This is particularly useful when testing authentication and authorization mechanisms.

### cURL provides a lightweight alternative

The same vulnerability could be demonstrated directly from the command line using cURL, without requiring a graphical penetration-testing tool.

### All temporary access methods should be removed 
temporary access methods like default passwords and access headers should be removed pre production

---

# 9. Methodology Summary

| Stage             | Action                               | Result                        |
| ----------------- | ------------------------------------ | ----------------------------- |
| Reconnaissance    | Accessed login page                  | Email known, password unknown |
| Source Analysis   | Inspected page source                | Discovered `Access` header    |
| Encoding Analysis | Identified ROT13                     | Header value decoded          |
| Exploitation 1    | Added header using cURL              | Authentication bypassed       |
| Exploitation 2    | Intercepted request using Burp Suite | Header successfully added     |
| Post-Exploitation | Accessed protected resource          | Flag captured                 |

---

# 10. Tools and Environment

### Kali Linux

Used for the command-line exploitation method with cURL.

### Windows

Used as the desktop operating system during browser-based testing.

### Brave Browser

Used to access the challenge application and inspect the page source.

### Burp Suite

Used to intercept and modify the login HTTP request.

### cURL

Used to manually construct and send the HTTP request containing the decoded `Access` header.

---


# Conclusion

The **Crack the Gate** challenge demonstrated how weak authentication mechanisms can be identified through basic web application reconnaissance.

The key discovery was an `Access` header hidden in the page source. The header value was encoded using ROT13, which was easily decoded. I then demonstrated the vulnerability using two different approaches.

First, I used **cURL on Kali Linux** to construct a request containing the decoded header. Second, I used **Burp Suite** to intercept the browser's login request, add the header manually, and forward the modified request.

Both approaches successfully bypassed the intended authentication mechanism and allowed the flag to be captured.

The main lesson from the challenge is that **obfuscation is not a substitute for security, and client-controlled HTTP headers should never be trusted as authentication credentials without proper server-side validation.**
