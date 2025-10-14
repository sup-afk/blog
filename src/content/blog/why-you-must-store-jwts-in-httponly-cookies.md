---
title: "🔒 Why you must store JWTs in HttpOnly Cookies"
description: ""
pubDate: "June 08 2025"
setup:
---

## Introduction: Token-Based Authentication

When developing a web application, **Authentication** is a necessary feature for identifying users. The popular modern approach is **Token-based Authentication**, specifically using **JSON Web Tokens (JWTs)**.

The process is simple:
1.  A user sends their credentials (like username and password) to the server.
2.  If the credentials are valid, the server creates a **JWT** and sends it back to the client (browser).
3.  For all subsequent requests, the client attaches this JWT to prove its identity.

### The Rookie Mistake: Local Storage ⚠️

Many beginner web developers choose to store the resulting JWT in **Local Storage**. It's convenient and easy to access using JavaScript.

**However,** storing the Token in Local Storage exposes it to a critical vulnerability: **Cross-Site Scripting (XSS)**.

> If your website has an XSS vulnerability, an attacker can inject malicious JavaScript code into your page. This code can then immediately **access and steal the Token** from the user's Local Storage, allowing the hacker to impersonate the user.

---

## The Secure Solution: HttpOnly Cookies 🍪

Since the primary security issue stems from the Token being accessible by client-side JavaScript, the solution is to move the storage to a location that is inaccessible by scripts: **HttpOnly Cookies**.

### What is an HttpOnly Cookie?

"HttpOnly" is a flag set on a cookie by the server. It has two crucial functions:
1.  **Inaccessible by JavaScript:** When a cookie is set with the "HttpOnly" flag, scripts running in the browser (e.g., via "document.cookie") **cannot read, write, or modify** that cookie.
2.  **Automatic Submission:** The browser automatically attaches the cookie to every HTTP request sent to the designated domain.

### Why HttpOnly is Superior for JWTs

- **XSS Protection:** Even if a webpage is successfully attacked via XSS, the malicious script **cannot steal the Token**, as it is hidden from JavaScript execution.
- **Convenience:** You don't need to write client-side code to pull the Token from storage and manually attach it to the request header; the browser handles the submission automatically.

---

## Implementing with Node.js 🛠️

Setting the "HttpOnly" flag is strictly a server-side operation.

When a user successfully logs in, your server must set the JWT within a cookie using the correct flags.

To set this up, you'll typically use these Node.js dependencies:
1.  "express": The framework for building the server.
2.  "cookie-parser": Middleware to handle cookies.
3.  "cors": Middleware for managing Cross-Origin Resource Sharing (necessary during development).
4.  "jsonwebtoken": For creating and verifying the JWT itself.

```javascript
// Example Server-Side Code (Node.js/Express)
// Assume 'token' is the generated JWT string

res.cookie("jwt", token, {
  httpOnly: true, // Set to HttpOnly to prevent JS access (Crucial!)
  secure: process.env.NODE_ENV === "production", // Use secure: true in production (requires HTTPS)
  sameSite: "strict", // Helps mitigate Cross-Site Request Forgery (CSRF)
  maxAge: 3600000, // Sets the cookie expiration to 1 hour (in milliseconds)
});

res.status(200).json({ message: "Login successful" });
```
