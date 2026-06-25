# JWT_Token

**1)** **What is JSON Web Token?**
JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.

**2) What is the JSON Web Token structure?**
In its compact form, JSON Web Tokens consist of three parts separated by dots (.), which are:
Header
Payload
Signature
Therefore, a JWT typically looks like the following: xxxxx.yyyyy.zzzzz
**Header:**
The Header contains metadata about the token.
Meaning
alg → Algorithm used to sign the token (HS256, RS256, ES256, etc.)
typ → Token type (JWT)
**Payload:**
The Payload contains the claims (information about the user).

| Claim | Meaning           |
| ----- | ----------------- |
| `sub` | Subject (User ID) |
| `iss` | Issuer            |
| `aud` | Audience          |
| `exp` | Expiration Time   |
| `iat` | Issued At         |
| `nbf` | Not Before        |
| `jti` | JWT ID            |

Important: The payload is not encrypted by default. It is only Base64URL encoded, so anyone can decode it. Never store sensitive information like passwords or OTPs in the payload.

**Signature:**

Signature =
HMACSHA256(
Header.Payload,
"MySecretKey"
)

The Signature ensures that the JWT has not been modified. **If anyone changes even one character in the Header or Payload, the signature changes, and verification fails.**
During authentication, the server verifies the signature to ensure the token hasn't been tampered with.

**3) Difference Between Validating and Verifying a JWT ?**

JWT validation generally refers to checking the structure, format, and content of the JWT:

Structure: Ensuring the token has the standard three parts (header, payload, signature) separated by dots.
Format: Verifying that each part is correctly encoded (Base64URL) and that the payload contains expected claims.
Content: Checking if the claims within the payload are correct, such as expiration time (exp), issued at (iat), not before (nbf), among others, to ensure the token isn't expired, isn't used before its time, etc.

JWT verification, on the other hand, involves confirming the authenticity and integrity of the token:

Signature Verification: This is the primary aspect of verification where the signature part of the JWT is checked against the header and payload. This is done using the algorithm specified in the header (like HMAC, RSA, or ECDSA) with a secret key or public key. If the signature doesn't match what's expected, the token might have been tampered with or is not from a trusted source.
Issuer Verification: Checking if the iss claim matches an expected issuer.
Audience Check: Ensuring the aud claim matches the expected audience.

**4) How do you invalidate a JWT token?**
A JWT is invalidated when the application no longer wants to honor the permissions or identity represented by that token. Common reasons include logout, password changes, account deactivation, and permission updates. Since JWTs are stateless, invalidation is typically achieved using token revocation lists (Token blacklisting), token versioning, short expiration times, etc...

**5) What is Token Blacklisting?**
Token blacklisting is a way to revoke a JWT before its expiration time.
General Steps:
User Login
    ↓
JWT Issued (expires in 1 hour)
    ↓
API validates (signature + expiry + Not Blacklisted)
    ↓
Access Granted

Even if the user logs out after 5 minutes, the JWT is still technically valid for the remaining 55 minutes. A blacklist solves this problem.
So, Token blacklisting is a JWT revocation mechanism. Each JWT contains a unique jti (JWT ID) claim. The jti is stored in a blacklist (commonly Redis) with a TTL equal to the token's remaining lifetime. When a user logs-out or when a token must be revoked,  So every request, after validating the JWT signature and expiration, the application checks whether the jti exists in the blacklist. If it does, the token is rejected even though it has not yet expired. Once the token's original expiration time is reached, the Redis entry expires automatically because of the configured TTL.

**6) Where can you store a JWT token on the client side, and what are the trade-offs?**

There are three client-side options and one server-side option:
**localStorage** is persistent browser storage written to disk. Any JavaScript on the page can read it via localStorage.getItem(), which makes it vulnerable to XSS attacks — if an attacker injects a script, they can steal the token instantly. It survives page refresh and tab close.

**httpOnly cookie** is also written to disk, but the browser automatically blocks JavaScript from reading it — only the server can access it. This makes it XSS-safe, but it's vulnerable to CSRF attacks (a malicious site can trick the browser into sending the cookie automatically). You mitigate CSRF with a SameSite=Strict or SameSite=Lax flag.

**In-memory (client-side)** means storing the token in a plain JavaScript variable — a let token = ... in your app. No script outside your own module can reach it, making it XSS-safe. But it's gone the moment the user refreshes the page, so you typically pair it with a httpOnly refresh token cookie to silently get a new access token on reload.

**Server-side in-memory** is what we use in our codebase. The token never reaches the browser at all — it lives as a module-level variable in the Node.js process memory on the server. The browser calls our Next.js API route, and that route internally fetches and uses the token. There is no client-side storage involved whatsoever. The only attack surface is the server process itself.

**#OAuth 2.0 — Authorization framework**

OAuth solves one problem: "How can I let a third-party app access my data without giving it my password?"
When you click "Login with Google" on some app, OAuth is what happens behind the scenes. Your app redirects you to Google's login page. You log in and click "Allow". Google sends back a short-lived one-time auth code to your app. Your app then exchanges that code (plus its own client secret) with Google's server to get an access token. That access token is what the app uses to call Google APIs on your behalf — like reading your calendar or profile.
The critical thing: OAuth's access token only says "this app is allowed to do X." It does not tell the app who you are.

