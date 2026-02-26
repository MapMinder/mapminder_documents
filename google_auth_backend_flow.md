# Backend authentication flow - Google sign in(MVP)

---

- [Sequence Diagram](#sequence-diagram)
- [Google ID Token Verification - Endpoint & Response](#google-id-token-verification---endpoint--response)
    - [Endpoint](#endpoint)
    - [response](#response)

---

## Sequence Diagram

```mermaid
sequenceDiagram
participant mf as app
participant mb as server
participant db as database
participant auth as OAuth Server

mf->>mb: POST /auth/google with id_token<br>(received from Flutter's Google_Sign_In SDK)
mb->>auth: verify id_token
auth->>auth: verify signature, aud, exp
alt valid token
    auth-->>mb: return claims (sub, email, name)
    mb->>mb: extract sub from claims
    mb->>db: check if user exists
    alt user does not exist
        mb->>db: create user and oauth record
        mb->>mb: generate JWT for user_id
    else user exists
        mb->>mb: generate JWT for user_id
    end
    mb-->>mf: return JWT (to be used in Authorization header for protected endpoints)
else invalid token
    auth-->>mb: returns error
    mb-->>mf: google sign in error
end
```

---

## Google ID Token Verification - Endpoint & Response

### Endpoint

> GET https://oauth2.googleapis.com/tokeninfo?id_token=<ID_TOKEN>

### response

**valid token response**

```json
{
  "iss": "https://accounts.google.com",
  "azp": "YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com",
  "aud": "YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com",
  "sub": "1078561234567890",
  "email": "user@example.com",
  "email_verified": "true",
  "name": "John Doe",
  "picture": "https://lh3.googleusercontent.com/a-/AOh14Gg1234567890",
  "given_name": "John",
  "family_name": "Doe",
  "locale": "en",
  "iat": "1686768000",
  "exp": "1686771600"
}
```

**invalid token response**

```json
{
  "error_description": "Invalid Value"
}
```
