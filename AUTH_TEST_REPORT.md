# Task 03 — Authentication and Authorization Test Report

Recorded UTC: 2026-10-23T10:48:03.380026+00:00.

**Blocked: 0 Pass, 0 Fail, 6 Not Run.** Four actual API requests were attempted (valid login, invalid login, missing token, invalid token); all failed at the proxy before an API response. The dependent valid-token and role-based cases were not sent. No results were simulated.

Base URL: https://dummyjson.com. Exactly six cases: three positive and three negative. No framework installed; curl uses TLS verification and a 20-second timeout. Credentials and tokens are redacted. Only public practice/demo data was attempted; no personal account credentials were used.

## Evidence and test oracle

The upstream [README](https://github.com/Ovi/DummyJSON/blob/master/README.md) was retrieved successfully (HTTP 200); it links an Auth resource to https://dummyjson.com/docs/auth. The live auth documentation request failed with curl exit 56, HTTP 000: `CONNECT tunnel failed, response 403`. This is an environment proxy denial, not an API authentication/authorization response. No origin response body or status was obtained.

Expected login/token outcomes below are explicitly provisional test hypotheses, not requirements verified from current API documentation. Demo credential validity is also not verified. Do not classify any authentication expectation as passed or failed until API access and documentation are available.

Role-based authorization support is **not established**. No documented role-protected route was verified; do not infer access control from a `role` field or invent an admin endpoint. If subsequent documentation confirms RBAC is unsupported, AUTH-03 remains Not Run with that limitation. It is not a passing role-authorization test.

## Results

| ID | Type | Scenario | Request details (redacted) | Expected result | Actual result / response summary | Actual API HTTP status | Result |
| --- | --- | --- | --- | --- | --- | --- | --- |
| AUTH-01 | Positive | Valid demo login | POST /auth/login; Content-Type: application/json; body: {"username":"[PUBLIC_DEMO_USER]","password":"[REDACTED]","expiresInMins":5} | Provisional: 200 and nonempty access token plus account identity. Verify current documentation and demo credentials first. | curl exit 56; origin HTTP unavailable (curl 000); curl: (56) CONNECT tunnel failed, response 403 | Unavailable | Not Run |
| AUTH-02 | Positive | Valid token | GET /auth/me; Authorization: Bearer [REDACTED_ACCESS_TOKEN_FROM_AUTH-01] | Provisional: 200; returned identity matches AUTH-01. | Not attempted: AUTH-01 could not reach API; no token obtained. | Unavailable | Not Run |
| AUTH-03 | Positive | Role-based allowed operation | Endpoint and authorized demo role: not established; do not invent an admin route. | If documented RBAC exists, an authorized role may access its documented protected operation. Otherwise record unsupported scope. | Not attempted: no verified role-protected endpoint or role contract. Role fields alone would not prove enforcement. | Unavailable | Not Run |
| AUTH-04 | Negative | Invalid login | POST /auth/login; Content-Type: application/json; deliberately nonexistent demo username and incorrect password [REDACTED] | Provisional: 4xx authentication rejection; no usable token; exact code/message requires documentation. | curl exit 56; origin HTTP unavailable (curl 000); curl: (56) CONNECT tunnel failed, response 403 | Unavailable | Not Run |
| AUTH-05 | Negative | Missing token | GET /auth/me; no Authorization header, cookies, or login session | Provisional: 401 authentication rejection; no protected account data. | curl exit 56; origin HTTP unavailable (curl 000); curl: (56) CONNECT tunnel failed, response 403 | Unavailable | Not Run |
| AUTH-06 | Negative | Invalid token | GET /auth/me; Authorization: Bearer [INVALID_SYNTHETIC_TOKEN] | Provisional: 401 authentication rejection; no protected account data. | curl exit 56; origin HTTP unavailable (curl 000); curl: (56) CONNECT tunnel failed, response 403 | Unavailable | Not Run |

## Findings and next step

No API defects or authorization guarantees were verified. The environment allowlist draft now includes `dummyjson.com`, preserving `jsonplaceholder.typicode.com`. Saving the draft does not apply network changes. Review and save the network change in environment settings, then publish the environment. Once access is applied, verify live documentation/demo credentials and rerun these same six cases; do not add cases or turn blocked cases into passes.

Task 01 and Task 02 files were left unchanged. No credentials, token values, or raw login responses are included in these reports.
