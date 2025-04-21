---
layout: post
title: "🛡️ From Login to Lockdown: Dissecting a Broken Authentication Flow"
author: Sahana Narasimha Murthy
date: 2025-04-12
tags: [OWASP, IDOR, Burp Suite, Broken Authentication, Web Security]
cover_image: /assets/images/cover-lockdown.png
description: A hands-on walkthrough of how I uncovered a broken access control vulnerability in a login flow using OWASP Juice Shop and Burp Suite.
permalink: /2025/04/12/from-login-to-lockdown/
---

## Introduction: Trust in Authentication and the Pitfall of Assumptions

Authentication is foundational to every secure application. It establishes identity, grants access, and builds the boundary between users and their data.

However, **authentication alone is not security**.  
What follows — *authorization* — is just as critical, and often overlooked.

This article explores a subtle but impactful security flaw:  
a scenario where an application authenticated me correctly but failed to verify whether I had the right to access what I was asking for.

The result?  
**Exposure of another user’s data.**  
This is known as **Insecure Direct Object Reference (IDOR)** — a vulnerability that may appear harmless on the surface but has serious implications when left unchecked.

---

## 🔍 Understanding IDOR — Insecure Direct Object Reference

**IDOR** occurs when an application exposes a way to access internal resources (like user profiles, orders, or files) through a direct reference such as an ID or any value — **without verifying whether the current user is authorized to access it.**

### 📌 Example

```
GET /user/**125**/profile
```
If you change that to:
```
GET /user/**126**/profile
```
...and the server returns someone else’s data — that's IDOR.

> In the OWASP Top 10, IDOR falls under [Broken Access Control](https://owasp.org/Top10/en/A01_2021-Broken_Access_Control/), which ranks as the most critical web application risk.

---

## 🧪 Objective of This Assessment

> **To determine whether I could access resources belonging to another user, despite being authenticated as myself.**

Using **OWASP Juice Shop** and **Burp Suite**, I analyzed authenticated traffic and attempted to manipulate object identifiers to observe the server’s access control behavior.

---

## ⚙️ The Flow I Followed

### 1. Authentication: Observing the Login Behavior

After logging in through Burp's embedded browser, I intercepted and observed:

```http
POST /rest/user/login
{
  "email": "sahana@test.com",
  "password": "test123"
}
```

Burp Suite allowed me to inspect this request and the resulting response. The application issued a JWT token which was then used in further communication.

### 2. Identity Confirmation via `/whoami`

Right after login, the application confirmed my identity by sending a token-authenticated request to:

```http
GET /rest/user/whoami
```

![Whoami Request](/LayerZero/assets/images/whoami-request.png)

This indicated a successful authentication and a session in progress.

---

### 3. Discovering a Vulnerable Endpoint

While exploring features like **Order History** and **My Basket**, I monitored HTTP history in Burp Suite and noticed:

```http
GET /rest/basket/6
```

![HTTP History Basket ID](/LayerZero/assets/images/http-history-basket-id.png)

This `6` appeared to be a direct object reference — possibly my current basket ID.

> I wanted to know: *What happens if I try to change the ID to basket 1 instead?*

---

### 4. Testing for IDOR via Repeater

I used Burp Repeater to modify the original request:

```http
GET /rest/basket/1
Authorization: Bearer <valid_token>
```

![Modified Repeater Request](/LayerZero/assets/images/repeater-modify-basket-id.png)

After sending the modified request:

![Repeater Response from Basket 1](/LayerZero/assets/images/repeater-response-basket-1.png)

The response returned details of **another user’s basket (User ID = 1)**, as you can see confirming that no access control check was in place.

---

## 🏢 Real-World IDOR Breaches in the U.S.

These weren’t just hypothetical risks — IDOR has played a role in several significant breaches in recent years:

### 1. **First American Financial Corp.**  
**Year**: 2019  
**Impact**: Exposed over 885 million records including mortgage documents, Social Security numbers, bank accounts, and tax records.  
**Cause**: An IDOR vulnerability allowed unauthenticated users to increment document IDs in the URL and access sensitive files.  
🔗 [Source – Forbes](https://www.forbes.com/sites/ajdellinger/2019/05/26/understanding-the-first-american-financial-data-leak-how-did-it-happen-and-what-does-it-mean/)

---

### 2. **Facebook (Meta)**  
**Year**: 2021  
**Impact**: Personal data of 533 million users leaked, including phone numbers, Facebook IDs, full names, and locations.  
**Cause**: An IDOR-style flaw in Facebook's contact importer tool allowed scraping user data without proper access control.  
🔗 [Source – NPR](https://www.npr.org/2021/04/09/986005820/after-data-breach-exposes-530-million-facebook-says-it-will-not-notify-users)

---

### 3. **Unnamed Major U.S. Bank**  
**Year**: 2023  
**Impact**: Exposure of PII including full names, email addresses, and offers tied to other users.  
**Cause**: An IDOR vulnerability in the bank’s offer-linking system allowed users to manipulate URLs and access others’ offers.  
🔗 [Source – bxmbn.medium.com](https://bxmbn.medium.com/i-received-a-bank-offer-in-my-mailbox-and-discovered-an-idor-vulnerability-5-000-bounty-bxmbn-5209cab1fba8?utm_source=chatgpt.com)

---

> These cases reinforce why Broken Access Control is ranked #1 in the [OWASP Top 10](https://owasp.org/Top10/en/A01_2021-Broken_Access_Control/). A single unchecked parameter can lead to a massive breach — regardless of the size or maturity of the company.

---

## 🧭 What If You Don’t See These Endpoints?

Not all endpoints like `/rest/basket/{id}` are immediately visible in UI actions. If you don’t find them in HTTP history, you can:

- Observe user ID from `/whoami`
- Try manually accessing `/user/{id}` or `/basket/{id}` through Repeater

> Testing IDOR is often about exploring assumptions, not brute force.

---

## 🛡️ Why This Is Dangerous

IDOR vulnerabilities allow attackers to:

- View or modify other users' sensitive data
- Download confidential records (e.g., invoices, orders)
- Perform actions on behalf of other users

This can lead to **privacy breaches**, **data leaks**, and **regulatory violations**.

---

##  How to Prevent IDOR Vulnerabilities

| Practice                            | Why it's important |
|-------------------------------------|---------------------|
| **Server-side ownership checks**    | Don’t rely on frontend validation |
| **Avoid predictable object IDs**    | Use UUIDs or indirect references |
| **Access control for every request**| Validate every operation’s permission |
| **Security testing**                | Regularly test for IDOR and related flaws |

---

## 🧪 Lab Setup – Try It Yourself

### Tools Required:
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload)

### Instructions:

```bash
# Download Juice Shop (standalone build)
wget https://github.com/juice-shop/juice-shop/releases/download/v15.3.1/juice-shop-standalone-15.3.1.tgz

# Extract and run
tar -xvzf juice-shop-standalone-15.3.1.tgz
cd juice-shop-standalone_15.3.1
node juice-shop.js
```

Access it via `http://localhost:3000`.

1. Open Burp Suite → Turn Proxy Intercept ON  
2. Visit Juice Shop in Burp’s browser  
3. Sign up, log in, and start observing HTTP history  
4. Find endpoints referencing IDs and modify them using Repeater  

---

## 💬 Final Thoughts

This test didn’t involve complex exploits, only curiosity and logic.  
When access control isn’t enforced properly, **even authenticated users can become unauthorized attackers.**

What made this test successful wasn’t just the tools — it was the question that started it all:

* Can I access what someone else owns? *


---

> *From root to reason — this is <span style="color:#007acc; font-weight:bold;">LayerZero</span>.*

