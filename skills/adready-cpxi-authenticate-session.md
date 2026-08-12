---
name: Establish and maintain a Digital Remedy Platform session
description: >-
  Authenticate against the Digital Remedy Platform (Kickstart) API, hold a session JWT, refresh it
  before it expires, and release it on completion. Every other skill in this repo depends on this one.
api: openapi/adready-cpxi-kickstart-openapi.yml
operations:
  - authenticateUser
  - refreshToken
  - parseJwtToken
  - removeJwtToken
  - getConfig
  - getVersion
generated: '2026-08-12'
method: generated
source: >-
  Grounded in openapi/adready-cpxi-kickstart-openapi.yml (harvested from
  https://platform.digitalremedy.com/v3/api-docs) plus live probes on 2026-08-12. The provider
  publishes no developer documentation; every operationId below was verified verbatim in the spec.
---

# Establish and maintain a Digital Remedy Platform session

## Before you start — read this

This is a **private first-party application backend**, not a public API. Digital Remedy publishes no
developer portal, no onboarding, no pricing and no terms of API use. There is **no self-service
sign-up**: credentials are provisioned by Digital Remedy through
<https://www.digitalremedy.com/contact-us/>.

Do not run this skill without an account that Digital Remedy has issued to you and permission to
automate against it. The public OpenAPI describes the surface; it does not grant access to it.

Base URL: `https://platform.digitalremedy.com` (the host `https://platform.adready.com` serves the
same application under the AdReady brand).

## Steps

1. **Check the service is up and note the build.**
   `GET /version` (`getVersion`) — anonymous, returns `{version, gitRevision, buildTime, api:{kickstart-api:{...}}}`.
   Record the version. The API is unversioned in its paths, so this build string is the only handle
   you have on what contract you are talking to. At the time of writing it reported `3.5.6`.

2. **Fetch client bootstrap configuration.** `GET /api/config` (`getConfig`) — also anonymous.
   Treat its contents as configuration for Digital Remedy's own client, not as a contract for you.
   It currently still advertises a legacy AdReady+ host whose DNS no longer resolves.

3. **Log in.** `POST /api/auth/login` (`authenticateUser`).
   - Required query parameter: `app` (integer). The spec does not enumerate valid values; use the
     value Digital Remedy gave you.
   - Body (`LoginRequest`): `{"email": "<string, format email>", "password": "<string>"}` — both required.
   - Two sibling login operations exist for other tenancies: `authenticatePlusUser`
     (`POST /api/auth/plus_login`, legacy AdReady+) and `loginAhUser`
     (`POST /api/auth/loginC360User`, Compulse 360 users). Use the one matching your account.
   - **The response is typed only as a bare `object` in the spec.** You must inspect the actual
     response and the `Set-Cookie` header to learn how the session is carried — the description does
     not tell you, because it declares no `securitySchemes` at all.

4. **Carry the session on every subsequent call.** The browser client uses axios with
   `withCredentials: true`, which is consistent with a cookie-borne session. Whatever the carrier,
   without it **every** business path returns `401` with an **empty body and no `WWW-Authenticate`
   header** — there is no machine-readable reason for the failure, so treat any `401` as "session
   absent or expired" and re-authenticate.

5. **Inspect the token when you need its claims.** `GET /api/token/parse` (`parseJwtToken`).

6. **Refresh before expiry.** `GET /api/token/refresh` (`refreshToken`) — returns
   `ApiResponseTokenWrapper`. The spec documents no token lifetime, so refresh proactively rather
   than waiting for a `401`.

7. **Release the session when done.** `DELETE /api/token/remove` (`removeJwtToken`).

## Rules this API imposes on you

- **No idempotency.** There is no `Idempotency-Key` anywhere in the description. **Never blindly retry
  a failed `POST` or `PUT`** — a timeout may or may not have applied. Re-read state first.
- **Read the envelope, not just the HTTP status.** Successful responses are
  `ApiResponse {status:int, message:string, result:T}`. `status` is an application-level code inside
  the body, so an HTTP `200` can carry a failure. Check both.
- **Errors carry almost no information.** Every operation declares only a generic `400` with an
  untyped `object` schema. This is not RFC 9457; there are no error type URIs to branch on.
- **No rate-limit signalling.** No `RateLimit-*` or `Retry-After` headers are returned and no `429` is
  declared. Undocumented limits may still exist behind auth — back off conservatively on your own
  schedule.
- **Never call the diagnostic endpoints** `GET /api/test/error` (`throwErr`) or `GET /throw_exception`
  (`throwException`). They exist only to raise errors and were left in the published description.

## References

- Authentication detail: `authentication/adready-cpxi-authentication.yml`
- Conventions: `conventions/adready-cpxi-conventions.yml`
- Errors: `errors/adready-cpxi-problem-types.yml`
