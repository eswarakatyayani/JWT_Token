# JWT_Token

1) What is JSON Web Token?
   JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.

2) What is the JSON Web Token structure?
In its compact form, JSON Web Tokens consist of three parts separated by dots (.), which are:
Header
Payload
Signature
Therefore, a JWT typically looks like the following: xxxxx.yyyyy.zzzzz

4) Difference Between Validating and Verifying a JWT ?

JWT validation generally refers to checking the structure, format, and content of the JWT:

Structure: Ensuring the token has the standard three parts (header, payload, signature) separated by dots.
Format: Verifying that each part is correctly encoded (Base64URL) and that the payload contains expected claims.
Content: Checking if the claims within the payload are correct, such as expiration time (exp), issued at (iat), not before (nbf), among others, to ensure the token isn't expired, isn't used before its time, etc.

JWT verification, on the other hand, involves confirming the authenticity and integrity of the token:

Signature Verification: This is the primary aspect of verification where the signature part of the JWT is checked against the header and payload. This is done using the algorithm specified in the header (like HMAC, RSA, or ECDSA) with a secret key or public key. If the signature doesn't match what's expected, the token might have been tampered with or is not from a trusted source.
Issuer Verification: Checking if the iss claim matches an expected issuer.
Audience Check: Ensuring the aud claim matches the expected audience.

5) How do you invalidate a JWT token?
A JWT is invalidated when the application no longer wants to honor the permissions or identity represented by that token. Common reasons include logout, password changes, account deactivation, and permission updates. Since JWTs are stateless, invalidation is typically achieved using token revocation lists (Token blacklisting), token versioning, short expiration times, etc...

6) What is Token Blacklisting?
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
So, Token blacklisting is a JWT revocation mechanism. Each JWT contains a unique jti claim. The jti is stored in a blacklist (commonly Redis) with a TTL equal to the token's remaining lifetime. When a user logs-out or when a token must be revoked,  So every request, after validating the JWT signature and expiration, the application checks whether the jti exists in the blacklist. If it does, the token is rejected even though it has not yet expired. Once the token's original expiration time is reached, the Redis entry expires automatically because of the configured TTL.
