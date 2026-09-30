# A01:2025 Broken Access Control — IDOR on Basket Endpoint

## Summary
The `/rest/basket/{id}` endpoint does not verify that the logged-in user
owns the basket being requested. Any authenticated user can view another
user's basket contents by changing the ID in the URL.

## Steps to reproduce
1. Registered and logged in as a test user (own basket = ID 6).
2. Captured the `GET /rest/basket/6` request in Burp Suite, including the
   `Authorization: Bearer <token>` header issued at login.
3. Sent the request to Burp Repeater and changed the URL to
   `/rest/basket/1`, keeping the same Authorization header unchanged.
4. Received `HTTP/1.1 200 OK` with basket 1's full contents (User ID 1),
   not the logged-in user's own basket.

## Evidence
- screenshots/a01-idor-basket-repeater.png
- evidence/a01-basket1-response.json

Basket 1 (belonging to UserId 1) returned:
- Apple Juice (1000ml) x2
- Orange Juice (1000ml) x3
- Eggfruit Juice (500ml) x1

Juice Shop's own scoring log confirmed this as a completed challenge:
`Restored 2-star basketAccessChallenge (View Basket)`

## Impact
Any authenticated user can enumerate basket IDs (1, 2, 3...) and read
other customers' basket contents. This exposes what products other users
have added, a privacy and business-logic issue. In a real store this
could reveal purchasing patterns or personal shopping behaviour.

## Root cause
The server checks that a valid Authorization token was supplied, but
never checks that the token's owner matches the requested basket ID.
It trusts the ID in the URL instead of the identity of the caller.

## Recommended fix
On every request to /rest/basket/{id}, the server should compare the
basket's owner (UserId) against the authenticated user's ID from the
token, and return 403 Forbidden if they don't match.

## OWASP mapping
A01:2025 - Broken Access Control (IDOR / missing object-level
authorization check)
