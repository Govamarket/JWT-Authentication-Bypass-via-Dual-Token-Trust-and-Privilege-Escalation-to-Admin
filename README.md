# JWT-Authentication-Bypass-via-Dual-Token-Trust-and-Privilege-Escalation-to-Admin

## Summary

The application accepts two authentication parameters (`userToken` and `token`) in the same request and inconsistently validates them. By modifying the `role` claim in one JWT, an attacker can escalate privileges from a normal user to an administrator. The system appears to trust client-side JWT payload data without enforcing consistent signature verification or a single authoritative token source.

## Affected Endpoint

```
POST /auth2
```

## Authentication Parameters

```
userToken=<JWT>
token=<JWT>
```

## Vulnerability Details

The application processes two JWTs in the same request but does not enforce a single trusted authentication source. This allows an attacker to:

- Modify the `role` field in one token
- Keep the second token unchanged
- Still obtain elevated privileges (administrator access)

This indicates broken authentication and inconsistent token validation logic.

## Steps to Reproduce

**1. Send original request:**

```
POST /auth2
userToken=<valid_user_jwt>&token=<valid_user_jwt>
```

**2. Modify JWT payload:**

Change:
```json
"role": "user"
```

To:
```json
"role": "admin"
```

**3. Replace only one parameter:**

```
userToken=<modified_admin_jwt>&token=<original_jwt>
```

**4. Send the modified request and observe the response.**

## Observed Result

- HTTP `200 OK` returned
- Response includes:

```
Set-Cookie: userRole=administrator
Set-Cookie: sessionId=admin_session_12345
```

- Admin-level access granted

## Impact

- Privilege escalation (user → admin)
- Authentication bypass
- Potential full account takeover
- Unauthorized access to administrative functions

**Severity: Critical**

## Root Cause

- Dual JWT parameters are accepted and inconsistently validated
- Trust is placed on client-controlled JWT `role` claim
- No strict enforcement of a single authentication source
- Possible missing or inconsistent signature verification

## Recommendations

- Use only one authentication token (via `Authorization` header)
- Enforce strict JWT signature verification on every request
- Never trust `role` or `isAdmin` claims sourced from client JWTs
- Remove duplicate token handling logic
- Validate roles server-side from the database

## Evidence Checklist

Before submitting, ensure the following are attached:

- [ ] Full raw HTTP request (Burp Suite copy)
- [ ] Full raw HTTP response showing admin cookies
- [ ] Before/after comparison of JWT payloads
- [ ] Screenshot of modified JWT (optional but recommended)
