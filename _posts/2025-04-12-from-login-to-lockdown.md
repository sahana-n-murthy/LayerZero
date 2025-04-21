---
layout: post
title: "From Login to Lockdown: Dissecting a Broken Authentication Flow"
author: "Sahana Narasimha Murthy"
date: 2025-04-12
tags: [OWASP, IDOR, Burp Suite, Broken Authentication, Web Security]
cover_image: /assets/images/cover-lockdown.png
permalink: /2025/04/12/from-login-to-lockdown/
---

<h2 style="color:#000; font-weight: bold;">
  From Login to Lockdown: Dissecting a Broken Authentication Flow
</h2>

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

> *From root to reason — this is <span style="color:#0366d6; font-weight:bold;">LayerZero</span>.*


