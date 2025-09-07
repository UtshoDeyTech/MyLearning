# Token-Based Authentication: Sign Out, Scalability, and Industry Practices

This document explains different approaches to handling **sign out** in token-based authentication (JWT + Refresh Tokens). It includes full lifecycle diagrams, storage locations, refresh logic, logout handling, and guidance for real-world scenarios.

---

## 🔑 Core Concepts

### Access Token (JWT)

* Signed token containing user claims (e.g., `userId`, `roles`).
* Short-lived (15 min – 1 hour).
* Can be verified statelessly.
* Cannot be revoked immediately unless tracked.

### Refresh Token

* Longer-lived token (hours–days).
* Used to generate new access tokens when JWT expires.
* Stored securely (DB or HttpOnly cookie).
* Enables continuous user sessions without re-login.

### The Sign Out Problem

* JWTs alone cannot be revoked immediately.
* Stolen JWTs remain valid until expiry.
* Extra mechanisms (refresh token DB, blacklisting, rotation) are required for instant invalidation.

---

## ⚙️ Industry-Standard Approaches

### 1. Short-lived Access + Refresh Tokens

**Flow Details:**

1. User logs in → server issues JWT + refresh token.
2. JWT used for API requests.
3. Near expiry, client sends refresh token → server issues new JWT.
4. Logout → server deletes refresh token from DB. JWT expires naturally.

**Diagram:**

```
+---------+        +--------+        +------+        +-------+
|  User   |        | Client |        |Server|        |  DB   |
+---------+        +--------+        +------+        +-------+
     | login req        |                 |                |
     |----------------->|                 |                |
     |                  | /login          |                |
     |                  |---------------->| generate JWT & refresh
     |                  |                 | store refresh -> DB
     |                  |                 |---------------->|
     |                  |<----------------| send JWT + refresh
     |  tokens issued   | JWT -> client   |                |
     |<-----------------| refresh -> cookie/DB             |
     | API request      |                 |                |
     |----------------->| send JWT       |                |
     |                  |---------------->| validate JWT   |
     |                  |                 |<----------------|
     |<-----------------| response OK     |                |
     | JWT expired soon |                 |                |
     | use refresh token|                 |                |
     |----------------->| send refresh    |                |
     |                  |---------------->| validate in DB |
     |                  |<----------------| issue new JWT  |
     |<-----------------| new JWT to client cookie        |
     | Logout           |                 |                |
     |----------------->| /logout         |                |
     |                  |---------------->| delete refresh |
     |                  |                 |---------------->|
```

**Pros:** Stateless, scalable, widely used.
**Cons:** JWT cannot be revoked immediately.
**Best Fit:** General web/mobile apps.

---

### 2. Blacklist / Token Store (Redis or Database)

**Flow Details:**

* Store JWT ID or token in Redis/DB.
* Validate both JWT signature **and existence in Redis**.
* Logout → remove token → immediate invalidation.

**Diagram:**

```
+---------+        +--------+        +------+        +-------+
|  User   |        | Client |        |Server|        | Redis |
+---------+        +--------+        +------+        +-------+
     | login req        |                 |               |
     |----------------->|                 |               |
     |                  | /login          |               |
     |                  |---------------->| generate JWT & refresh
     |                  |                 | store token ID -> Redis
     |                  |                 |---------------->|
     |                  |<----------------| JWT + refresh sent
     | API request      |                 |               |
     |----------------->| send JWT       |               |
     |                  |---------------->| validate JWT & check Redis
     |                  |                 |<----------------|
     |<-----------------| response OK     |               |
     | Logout           |                 |               |
     |----------------->| /logout         |               |
     |                  |---------------->| remove token ID
     |                  |                 |---------------->|
```

**Pros:** Immediate revocation, session control.
**Cons:** Adds state, Redis/DB dependency.
**Best Fit:** Banking, healthcare, enterprise apps.

---

### 3. Rotating Refresh Tokens

**Flow Details:**

* Each refresh token use invalidates the old token.
* New JWT + refresh token issued.
* Old refresh token reuse → rejected (replay attack prevention).

**Diagram:**

```
Login:
Client ---------> Server
       generate JWT + refresh A stored in DB

Refresh Request:
Client ---------> Server (refresh A)
Server: validate refresh A
Server: invalidate refresh A, store refresh B
Server: issue new JWT

Replay Attack Attempt:
Attacker -------> Server (refresh A)
Server: reject request (token invalidated)
```

**Pros:** Strong protection against stolen refresh tokens.
**Cons:** More complex token lifecycle.
**Best Fit:** Mobile apps, long-lived sessions.

---

### 4. Opaque Tokens + Central Introspection

**Flow Details:**

* Server issues random opaque token.
* Each request: server validates token in DB/Redis.
* Logout → token deleted → immediate invalidation.

**Diagram:**

```
+---------+        +--------+        +------+
| Client  |        | Server |        |  DB  |
+---------+        +--------+        +------+
     | login req        |                 |
     |----------------->| generate opaque token
     |                  | store token -> DB
     |                  |---------------->|
     |<-----------------| opaque token to client
     | API request      |                 |
     |----------------->| check token in DB
     |                  |---------------->|
     |                  |<----------------|
     |<-----------------| response OK
     | Logout           |                 |
     |----------------->| delete token from DB
     |                  |---------------->|
```

**Pros:** Full control, immediate revocation.
**Cons:** DB/Redis hit on every request.
**Best Fit:** Enterprise microservices, auditing-required apps.

---

### 5. Sliding Expiration (Sliding JWT/Refresh Tokens)

**Flow Details:**

* Token expiry extended on every valid request.
* Keeps session alive as long as user is active.
* Refresh token used only when JWT is near original expiry.

**Diagram:**

```
Client request:
Client -----> Server: JWT near expiry
Server: issue new JWT with extended expiry
Client uses new JWT for subsequent requests
Logout: delete refresh token from DB
```

**Pros:** Smooth UX, long sessions.
**Cons:** Extends attack window if stolen.
**Best Fit:** SaaS apps requiring long, active sessions.

---

### 6. JWT with Revocation List (`jti` in DB)

**Flow Details:**

* Each JWT has a unique ID (`jti`).
* Server stores invalidated `jti`s.
* Requests validate JWT + check revocation list.
* Logout → add `jti` to list.

**Diagram:**

```
Client sends JWT (jti: 123)
Server: validate JWT signature
Server: check if jti 123 in revocation list
If found → reject request
```

**Pros:** Immediate revocation of JWTs.
**Cons:** Adds server-side state, cleanup required.
**Best Fit:** High-security apps needing per-device logout.

---

### 7. JWT with Embedded Session State

**Flow Details:**

* JWT contains minimal claims.
* Server stores session state (roles, permissions).
* Sensitive requests validate JWT + session state in DB.

**Diagram:**

```
Client -----> Server: JWT
Server: verify JWT signature
Server: check roles/permissions in DB
Authorize request based on session state
```

**Pros:** Stateless JWT + fine-grained control.
**Cons:** DB lookups reduce scalability.
**Best Fit:** Enterprise apps with dynamic role/permission updates.

---

### 8. JWT with Multiple Scopes / Audience Segmentation

**Flow Details:**

* Issue separate JWTs for different scopes/audiences.
* Each token has its own expiry and revocation rules.

**Diagram:**

```
Client receives:
- JWT API (scope: api)
- JWT Admin (scope: admin)
Each used only for respective endpoints
```

**Pros:** Fine-grained access, limits stolen token impact.
**Cons:** More token management.
**Best Fit:** Large SaaS apps with multiple APIs and user roles.

---

### 9. Short-lived JWT + Refresh Token Rotation + Device Binding

**Flow Details:**

* Each refresh token tied to a device/browser.
* Rotation + device binding ensures token theft on another device fails.

**Diagram:**

```
Device A: refresh token stored -> DB
Device B: stolen refresh token attempt
Server: device mismatch → reject request
```

**Pros:** Maximum security, prevents device impersonation.
**Cons:** Complex storage/logic.
**Best Fit:** Banking, finance apps, high-security multi-device apps.

---

### 10. Single Sign-On (SSO) / OpenID Connect Tokens

**Flow Details:**

* Tokens issued by identity provider (OIDC/OAuth2).
* Provider handles rotation, revocation, and session management.
* Client uses access tokens and ID tokens for APIs.

**Diagram:**

```
User login → Identity Provider → Access Token + ID Token
Client API request → Server validates token with provider
Logout → provider revokes tokens
```

**Pros:** Reduced custom auth logic, supports federated login.
**Cons:** Dependent on third-party provider, some latency.
**Best Fit:** Enterprise apps needing SSO, multi-service authentication.

---

## 🧭 Choosing the Right Approach

| Approach                          | Security  | Scalability | Complexity | Best Fit                             |
| --------------------------------- | --------- | ----------- | ---------- | ------------------------------------ |
| 1. Short-lived JWT + Refresh      | Medium    | High        | Low        | Web/Mobile apps                      |
| 2. Blacklist / Redis              | High      | Medium      | Medium     | Banking, enterprise apps             |
| 3. Rotating Refresh               | High      | Medium      | Medium     | Mobile, long-lived sessions          |
| 4. Opaque Tokens                  | High      | Medium      | Medium     | Microservices, auditing              |
| 5. Sliding Expiration             | Medium    | High        | Medium     | SaaS, long-active sessions           |
| 6. JWT + Revocation List          | High      | Medium      | Medium     | High-security web apps               |
| 7. JWT + Session State            | High      | Medium      | Medium     | Enterprise apps, dynamic permissions |
| 8. Multiple Scopes JWT            | Medium    | High        | Medium     | SaaS, multiple APIs                  |
| 9. JWT + Refresh + Device Binding | Very High | Medium      | High       | Banking, multi-device apps           |
| 10. SSO / OIDC                    | High      | High        | Low        | Enterprise, SSO apps                 |

---

## ✅ Best Practices

* Keep access tokens **short-lived** (≤1h).
* Store refresh tokens securely (HttpOnly cookies, encrypted DB).
* Use **refresh token rotation** to prevent replay attacks.
* Use Redis/blacklist for **instant revocation**.
* Enforce **HTTPS** and secure token storage.
* Consider device binding or multiple scopes for high-security apps.

---

This README now provides a **complete industry-standard guide**, covering **all common approaches**, full token lifecycles, storage, refresh logic, logout handling, and a **summary table for comparison**.

