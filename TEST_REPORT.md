# API Test Report

Run time (UTC): 2026-10-20T06:47:33.755975+00:00

Base URL: https://jsonplaceholder.typicode.com

Source: README.md. Executed real HTTPS requests using curl with TLS verification enabled, a 25-second per-request timeout, no retries, and no installed dependencies. Writes use synthetic test data against the public simulated API.

## Results

Total: 32; Pass: 31; Fail: 1; Not Run: 0.

## Test oracle and scope

The README names endpoints and coverage areas but specifies no validation contract. Conventional JSONPlaceholder resource counts, field schemas, and successful status codes are explicit test expectations here. Missing-resource GETs are expected to return 404; unmatched comment filters are expected to return an empty list. These are baseline expectations, not requirements quoted from README. Missing fields, empty bodies, invalid data types, and missing-target writes are characterization checks: acceptance is recorded as an observation, not a validation defect. A Pass for these checks means a non-5xx JSON response was obtained and recorded; it does not certify business validation. Malformed JSON is expected to return 400; its error content type is recorded without assuming a JSON error schema.

POST/PUT/PATCH/DELETE responses are simulated; successful writes do not establish persistence. The original post is compared before and after the write sequence. No authentication, performance, security, or exhaustive schema testing was claimed. Transport failures are Not Run, even when a request was attempted.

## Defects and observations

Verified defect **DEF-01 — Malformed JSON produces HTTP 500 and discloses a parser stack trace** (TC-30). Suggested severity: Medium; error handling is affected, while valid requests continue to work. This is an observed upstream API defect, not a local repository defect. Classification is based on malformed request syntax being a client error, not on undocumented business-field validation.

- Reproduction: send `POST /posts` with `Content-Type: application/json` and the exact raw body `{"title":`.
- Expected: HTTP 400 with a controlled client-error response. No particular JSON error schema is assumed.
- Actual: HTTP 500, `text/html; charset=utf-8`, and `SyntaxError: Unexpected end of JSON input`, followed by internal `/app/node_modules/body-parser/...` paths and a Node.js stack trace.
- Verification: reproduced twice total; both requests returned 500 with the same parser error. A subsequent valid POST control returned 201 and echoed all supplied fields with id=101.
- Impact: client syntax errors are misclassified as server failures and expose implementation details. This is separate from simulated write persistence.
- Request count: 32 primary test requests plus 2 diagnostic requests, 34 total. Diagnostic requests do not change the primary case totals.

```sh
curl --silent --show-error --max-time 25 --include \
  --request POST 'https://jsonplaceholder.typicode.com/posts' \
  --header 'Content-Type: application/json' \
  --data-binary '{"title":'
```

- TC-25 (Missing title): HTTP 201; Content-Type: application/json; charset=utf-8; {"body": "body", "userId": 1, "id": 101}
- TC-26 (Missing all fields): HTTP 201; Content-Type: application/json; charset=utf-8; {"id": 101}
- TC-27 (Invalid field types): HTTP 201; Content-Type: application/json; charset=utf-8; {"title": 123, "body": false, "userId": "invalid", "id": 101}
- TC-28 (Empty field values): HTTP 201; Content-Type: application/json; charset=utf-8; {"title": "", "body": "", "userId": 0, "id": 101}
- TC-29 (Empty raw request body): HTTP 201; Content-Type: application/json; charset=utf-8; {"id": 101}
- TC-31 (Patch nonexistent post): HTTP 200; Content-Type: application/json; charset=utf-8; {"title": "Missing target"}
- TC-32 (Delete nonexistent post): HTTP 200; Content-Type: application/json; charset=utf-8; {}

Full response bodies, headers, and transport errors are retained in `/workspace/scratch/api-testing-evidence.json`. No test was replaced with a mocked response.

## Detailed results

| ID | Scenario | Method / Endpoint | Test data | Expected result | Actual result | Status | Defect / Observation |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TC-01 | List posts | GET /posts | — | 200; JSON array of 100 posts, each with id, userId, title, body; JSON Content-Type. | HTTP 200; Content-Type: application/json; charset=utf-8; array length=100; first item={"userId": 1, "id": 1, "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit", "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"} | Pass |  |
| TC-02 | Read first post | GET /posts/1 | — | 200; JSON object with id=1, integer userId, string title/body; JSON Content-Type. | HTTP 200; Content-Type: application/json; charset=utf-8; {"userId": 1, "id": 1, "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit", "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"} | Pass |  |
| TC-03 | Read upper existing post boundary | GET /posts/100 | — | 200; JSON post id=100. | HTTP 200; Content-Type: application/json; charset=utf-8; {"userId": 10, "id": 100, "title": "at nam consequatur ea labore ea harum", "body": "cupiditate quo est a modi nesciunt soluta\nipsa voluptas error itaque dicta in\nautem qui minus magnam et distinctio eum\naccusamus ratione error aut"} | Pass |  |
| TC-04 | Read invalid/missing post ID 0 | GET /posts/0 | — | 404; JSON error/empty object for an unavailable post; no server failure. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-05 | Read invalid/missing post ID -1 | GET /posts/-1 | — | 404; JSON error/empty object for an unavailable post; no server failure. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-06 | Read invalid/missing post ID 101 | GET /posts/101 | — | 404; JSON error/empty object for an unavailable post; no server failure. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-07 | Read invalid/missing post ID 999999 | GET /posts/999999 | — | 404; JSON error/empty object for an unavailable post; no server failure. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-08 | Read invalid/missing post ID abc | GET /posts/abc | — | 404; JSON error/empty object for an unavailable post; no server failure. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-09 | Read invalid/missing post ID 1.5 | GET /posts/1.5 | — | 404; JSON error/empty object for an unavailable post; no server failure. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-10 | Create post | POST /posts | {"title": "API Testing", "body": "Testing API using Codex", "userId": 1} | 201; JSON echoes supplied fields and returns an integer id; simulated creation. | HTTP 201; Content-Type: application/json; charset=utf-8; {"title": "API Testing", "body": "Testing API using Codex", "userId": 1, "id": 101} | Pass | Simulated write; persistence checked separately. |
| TC-11 | Replace post | PUT /posts/1 | {"id": 1, "title": "Updated title", "body": "Updated body", "userId": 1} | 200; JSON contains id=1 and supplied fields; simulated update. | HTTP 200; Content-Type: application/json; charset=utf-8; {"id": 1, "title": "Updated title", "body": "Updated body", "userId": 1} | Pass | Simulated write; persistence checked separately. |
| TC-12 | Patch title | PATCH /posts/1 | {"title": "Patched title"} | 200; JSON id=1 with changed title and retained body/userId; simulated update. | HTTP 200; Content-Type: application/json; charset=utf-8; {"userId": 1, "id": 1, "title": "Patched title", "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"} | Pass | Simulated write; persistence checked separately. |
| TC-13 | Delete post | DELETE /posts/1 | — | 200; empty JSON object; simulated deletion. | HTTP 200; Content-Type: application/json; charset=utf-8; {} | Pass | Simulated write; persistence checked separately. |
| TC-14 | Read after simulated writes | GET /posts/1 | — | 200; original post remains unchanged because writes are simulated. | HTTP 200; Content-Type: application/json; charset=utf-8; {"userId": 1, "id": 1, "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit", "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"} | Pass | Original resource unchanged after POST/PUT/PATCH/DELETE. |
| TC-15 | List users | GET /users | — | 200; JSON array of 10 users with id/name/username/email/address/phone/website/company. | HTTP 200; Content-Type: application/json; charset=utf-8; array length=10; first item={"id": 1, "name": "Leanne Graham", "username": "Bret", "email": "Sincere@april.biz", "address": {"street": "Kulas Light", "suite": "Apt. 556", "city": "Gwenborough", "zipcode": "92998-3874", "geo": {"lat": "-37.3159", "lng": "81.1496"}}, "phone": "1-770-736-8031 x56442", "website": "hildegard.org", "company": {"name": "Romaguera-Crona", "catchPhrase": "Multi-layered client-server neural-net", "bs": "harness real-time e-markets"}} | Pass |  |
| TC-16 | Read first user | GET /users/1 | — | 200; JSON user id=1 with name, username, email, address, phone, website, company. | HTTP 200; Content-Type: application/json; charset=utf-8; {"id": 1, "name": "Leanne Graham", "username": "Bret", "email": "Sincere@april.biz", "address": {"street": "Kulas Light", "suite": "Apt. 556", "city": "Gwenborough", "zipcode": "92998-3874", "geo": {"lat": "-37.3159", "lng": "81.1496"}}, "phone": "1-770-736-8031 x56442", "website": "hildegard.org", "company": {"name": "Romaguera-Crona", "catchPhrase": "Multi-layered client-server neural-net", "bs": "harness real-time e-markets"}} | Pass |  |
| TC-17 | Read upper existing user boundary | GET /users/10 | — | 200; JSON user id=10. | HTTP 200; Content-Type: application/json; charset=utf-8; {"id": 10, "name": "Clementina DuBuque", "username": "Moriah.Stanton", "email": "Rey.Padberg@karina.biz", "address": {"street": "Kattie Turnpike", "suite": "Suite 198", "city": "Lebsackbury", "zipcode": "31428-2261", "geo": {"lat": "-38.2386", "lng": "57.2232"}}, "phone": "024-648-3804", "website": "ambrose.net", "company": {"name": "Hoeger LLC", "catchPhrase": "Centralized empowering task-force", "bs": "target end-to-end models"}} | Pass |  |
| TC-18 | Read invalid/missing user ID 0 | GET /users/0 | — | 404; JSON error/empty object for an unavailable user. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-19 | Read invalid/missing user ID 11 | GET /users/11 | — | 404; JSON error/empty object for an unavailable user. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-20 | Read invalid/missing user ID abc | GET /users/abc | — | 404; JSON error/empty object for an unavailable user. | HTTP 404; Content-Type: application/json; charset=utf-8; {} | Pass |  |
| TC-21 | Filter comments for post 1 | GET /comments?postId=1 | — | 200; JSON array of 5 comments; every postId=1 with id/name/email/body. | HTTP 200; Content-Type: application/json; charset=utf-8; array length=5; first item={"postId": 1, "id": 1, "name": "id labore ex et quam laborum", "email": "Eliseo@gardner.biz", "body": "laudantium enim quasi est quidem magnam voluptate ipsam eos\ntempora quo necessitatibus\ndolor quam autem quasi\nreiciendis et nam sapiente accusantium"} | Pass |  |
| TC-22 | Filter comments for absent/invalid post 0 | GET /comments?postId=0 | — | 200; empty JSON array for an unmatched filter. | HTTP 200; Content-Type: application/json; charset=utf-8; [] | Pass |  |
| TC-23 | Filter comments for absent/invalid post 101 | GET /comments?postId=101 | — | 200; empty JSON array for an unmatched filter. | HTTP 200; Content-Type: application/json; charset=utf-8; [] | Pass |  |
| TC-24 | Filter comments for absent/invalid post abc | GET /comments?postId=abc | — | 200; empty JSON array for an unmatched filter. | HTTP 200; Content-Type: application/json; charset=utf-8; [] | Pass |  |
| TC-25 | Missing title | POST /posts | {"body": "body", "userId": 1} | Characterization: record status/body and field handling; no undocumented required-field/type validation assumed. Response must be valid JSON without server failure. | HTTP 201; Content-Type: application/json; charset=utf-8; {"body": "body", "userId": 1, "id": 101} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
| TC-26 | Missing all fields | POST /posts | {} | Characterization: record status/body and field handling; no undocumented required-field/type validation assumed. Response must be valid JSON without server failure. | HTTP 201; Content-Type: application/json; charset=utf-8; {"id": 101} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
| TC-27 | Invalid field types | POST /posts | {"title": 123, "body": false, "userId": "invalid"} | Characterization: record status/body and field handling; no undocumented required-field/type validation assumed. Response must be valid JSON without server failure. | HTTP 201; Content-Type: application/json; charset=utf-8; {"title": 123, "body": false, "userId": "invalid", "id": 101} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
| TC-28 | Empty field values | POST /posts | {"title": "", "body": "", "userId": 0} | Characterization: record status/body and field handling; no undocumented required-field/type validation assumed. Response must be valid JSON without server failure. | HTTP 201; Content-Type: application/json; charset=utf-8; {"title": "", "body": "", "userId": 0, "id": 101} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
| TC-29 | Empty raw request body | POST /posts | '' | Characterization: record status/body; no undocumented empty-body rejection assumed. Response must be valid JSON without server failure. | HTTP 201; Content-Type: application/json; charset=utf-8; {"id": 101} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
| TC-30 | Malformed JSON error handling | POST /posts | '{"title":' | 400 client error for syntactically malformed JSON; record error format. | HTTP 500; Content-Type: text/html; charset=utf-8; SyntaxError: Unexpected end of JSON input<br>    at JSON.parse (<anonymous>)<br>    at parse (/app/node_modules/body-parser/lib/types/json.js:89:19)<br>    at /app/node_modules/body-parser/lib/read.js:121:18<br>    at invokeCallback (/app/node_modules/body-parser/node_modules/raw-body/index.js:224:16)<br>    at done (/app/node_modules/body-parser/node_modules/raw | Fail | DEF-01: reproduced HTTP 500 with parser stack trace; valid POST control returned 201. |
| TC-31 | Patch nonexistent post | PATCH /posts/999999 | {"title": "Missing target"} | Characterization: record missing-target status/body; do not assume existence checks for simulated writes. No server failure. | HTTP 200; Content-Type: application/json; charset=utf-8; {"title": "Missing target"} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
| TC-32 | Delete nonexistent post | DELETE /posts/999999 | — | Characterization: record missing-target behavior; idempotent/simulated deletion may succeed. No server failure. | HTTP 200; Content-Type: application/json; charset=utf-8; {} | Pass | Observation: characterization only; acceptance of missing/invalid fields or nonexistent write targets is not a verified defect. |
