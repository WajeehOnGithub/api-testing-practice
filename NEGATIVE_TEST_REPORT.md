# Advanced Negative and Boundary Test Report

Execution timestamp (UTC): 2026-10-22T05:52:14.987784+00:00. Base URL: https://jsonplaceholder.typicode.com. Inputs: README.md, TEST_CASES.md, TEST_REPORT.md. Task 01 reports are unchanged.

49 primary cases: **46 Pass, 3 Fail, 0 Not Run**. Three additional targeted reproductions: 52 curl requests total. Every primary case received an HTTP response with curl exit 0. No framework or dependency was installed.

## Interpretation and limits

The README defines routes and test scope, but no required fields, maximum lengths, type constraints, media-type policy, or unsupported-method status contract. Most cases characterize behavior: Pass means a real response satisfied the stated generic robustness check (2xx–4xx, no 5xx), not that the input was correctly rejected or that production validation is adequate. Fail indicates a violated explicit test expectation; it does not automatically establish a business-rule defect. Client-input parser failures returning 500 with implementation stack traces are reproducible error-handling findings. Top-level `null` is valid JSON; rejecting it is not itself a defect.

GET controls check status, JSON Content-Type and id=1; the before/after complete bodies match. The valid POST control checks status, echoed fields and integer id. Filter cases check that 200 responses contain arrays; all tested unmatched/empty filters returned `[]`. ID status outcomes are observations, without requiring a specific 400-versus-404 policy. Unsupported methods are outside README; responses may originate from an intermediary, and do not prove the application handled them. Headers and error bodies are retained as evidence without inventing mandatory CORS/security/header requirements.

Writes are simulated and not persistent. Acceptance of missing/null/wrong-type fields, empty objects, or unsupported media types is an observation. Lengths 255, 256, and 10,000 are exploratory samples, not documented limits; no maximum-size boundary was established. Tests are sequential, with TLS verification enabled, a 20-second timeout, no automatic retries or redirects, and no load testing. Transport failures would be Not Run, never Pass.

## Verified error-handling findings

These findings concern observed server-side error responses, not undocumented field-validation requirements. No source code was inspected, so exact root cause and remediation remain upstream responsibilities.

| Finding | Cases | Verified evidence | Classification |
| --- | --- | --- | --- |
| NEG-F01 | NEG-47 | Malformed raw JSON `{"title":` returned 500 twice, with `SyntaxError: Unexpected end of JSON input` and internal module paths. Valid POST control returned 201. | Reproduced Task 01 DEF-01: error handling/stack disclosure. A malformed-input client error should be controlled rather than exposed as an internal failure. |
| NEG-F02 | NEG-45 | `application/json; charset=invalid-charset` returned 500 twice with `UnsupportedMediaTypeError: unsupported charset "INVALID-CHARSET"` and Express/body-parser stack. | Verified error handling/stack disclosure; rejecting an unsupported charset is reasonable, but the specific business contract does not prescribe 415. |
| NEG-F03 | NEG-29 | Top-level body `null` returned 500 twice with a strict-parser `SyntaxError` and internal module paths. Explicit null-valued fields returned 201. | Verified robustness/stack disclosure; no defect is claimed for declining a top-level primitive. `null` is valid JSON, so the reported parser error does not mean JSON syntax was malformed. |

All three violate the declared no-5xx robustness expectation. Suggested triage severity: Medium. They may share error middleware and are not asserted to be three independent root causes. Responses contain application parser/Express frames, supporting upstream application attribution rather than a local curl transport failure. These are upstream observations; no defect in this documentation-only repository was established.

## Other observations

- All tested missing/invalid IDs, including leading-zero `01`, returned 404 `{}`. Empty and invalid comment filters returned 200 `[]`.
- Empty objects, omitted fields, explicit null fields, wrong field types, and the top-level array were accepted by simulated writes. These are not confirmed validation defects.
- All three long titles returned 201 and were echoed exactly, including 10,000 characters. No truncation was observed and no maximum limit is claimed.
- TRACE returned HTML 405 with a Cloudflare marker; PROPFIND returned 404 `{}`; BREW returned 400 with an empty body. This records reachable-edge behavior; application-level method rejection is not established for TRACE/BREW.
- `text/plain`, `application/xml`, malformed media type `invalid`, and an absent Content-Type returned 201 with only `id=101`; the supplied JSON fields were not echoed. No documented 415 requirement exists.
- Final GET /posts/1 matched the initial complete body, consistent with simulated nonpersistent writes.

## Detailed results

| ID | Scenario | Method / endpoint | Request body | Request Content-Type | Expected result | HTTP status | Actual result | Result | Observations |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| NEG-01 | Valid read control | GET /posts/1 | (no body) | (absent) | 200; JSON post id=1; application/json Content-Type; before/after bodies equal. | 200 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "userId": 1,<br>  "id": 1,<br>  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",<br>  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-02 | posts ID 0 | GET /posts/0 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-03 | posts ID -1 | GET /posts/-1 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-04 | posts ID 101 | GET /posts/101 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-05 | posts ID 999999 | GET /posts/999999 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-06 | posts ID 9007199254740992 | GET /posts/9007199254740992 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-07 | posts ID abc | GET /posts/abc | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-08 | posts ID 1.5 | GET /posts/1.5 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-09 | posts ID %40%23%24 | GET /posts/%40%23%24 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-10 | posts ID %20 | GET /posts/%20 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-11 | posts ID %2F | GET /posts/%2F | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-12 | posts ID 01 | GET /posts/01 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-13 | users ID 0 | GET /users/0 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-14 | users ID -1 | GET /users/-1 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-15 | users ID 11 | GET /users/11 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-16 | users ID abc | GET /users/abc | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-17 | users ID %40%23%24 | GET /users/%40%23%24 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-18 | Comment filter postId=0 | GET /comments?postId=0 | (no body) | (absent) | Characterization: 2xx–4xx without 5xx; if 200, JSON list and all results match supplied filter (empty filter behavior observed only). | 200 | Response Content-Type: application/json; charset=utf-8; body: [] | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-19 | Comment filter postId=-1 | GET /comments?postId=-1 | (no body) | (absent) | Characterization: 2xx–4xx without 5xx; if 200, JSON list and all results match supplied filter (empty filter behavior observed only). | 200 | Response Content-Type: application/json; charset=utf-8; body: [] | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-20 | Comment filter postId=101 | GET /comments?postId=101 | (no body) | (absent) | Characterization: 2xx–4xx without 5xx; if 200, JSON list and all results match supplied filter (empty filter behavior observed only). | 200 | Response Content-Type: application/json; charset=utf-8; body: [] | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-21 | Comment filter postId=abc | GET /comments?postId=abc | (no body) | (absent) | Characterization: 2xx–4xx without 5xx; if 200, JSON list and all results match supplied filter (empty filter behavior observed only). | 200 | Response Content-Type: application/json; charset=utf-8; body: [] | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-22 | Comment filter postId=%40%23%24 | GET /comments?postId=%40%23%24 | (no body) | (absent) | Characterization: 2xx–4xx without 5xx; if 200, JSON list and all results match supplied filter (empty filter behavior observed only). | 200 | Response Content-Type: application/json; charset=utf-8; body: [] | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-23 | Comment filter postId= | GET /comments?postId= | (no body) | (absent) | Characterization: 2xx–4xx without 5xx; if 200, JSON list and all results match supplied filter (empty filter behavior observed only). | 200 | Response Content-Type: application/json; charset=utf-8; body: [] | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-24 | Empty object POST | POST /posts | '{}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-25 | Empty object PUT | PUT /posts/1 | '{}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 200 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 1<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-26 | Empty object PATCH | PATCH /posts/1 | '{}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 200 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "userId": 1,<br>  "id": 1,<br>  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",<br>  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-27 | Zero-byte JSON body | POST /posts | '' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-28 | Explicit null fields | POST /posts | '{"title": null, "body": null, "userId": null}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "title": null,<br>  "body": null,<br>  "userId": null,<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-29 | Top-level null | POST /posts | 'null' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 500 | Response Content-Type: text/html; charset=utf-8; body: SyntaxError: Unexpected token 'n', "#" is not valid JSON<br>    at JSON.parse (<anonymous>)<br>    at createStrictSyntaxError (/app/node_modules/body-parser/lib/types/json.js:158:10)<br>    at parse (/app/node_modules/body-parser/lib/types/json.js:83:15)<br>    at /app/node_modules/body-pars ... [truncated; 766 UTF-8 bytes total] | Fail | Robustness failure; upstream attribution/root cause requires review. Do not infer undocumented validation requirements. Reproduced once more with HTTP 500; see findings. |
| NEG-30 | Missing title | POST /posts | '{"body": "Synthetic QA data", "userId": 1}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "body": "Synthetic QA data",<br>  "userId": 1,<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-31 | Missing body | POST /posts | '{"title": "Negative test", "userId": 1}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "title": "Negative test",<br>  "userId": 1,<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-32 | Missing userId | POST /posts | '{"title": "Negative test", "body": "Synthetic QA data"}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "title": "Negative test",<br>  "body": "Synthetic QA data",<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-33 | Numeric and boolean text fields, string userId | POST /posts | '{"title": 123, "body": false, "userId": "abc"}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "title": 123,<br>  "body": false,<br>  "userId": "abc",<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-34 | Nested object and array field types | POST /posts | '{"title": {"x": 1}, "body": ["x"], "userId": [1]}' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "title": {<br>    "x": 1<br>  },<br>  "body": [<br>    "x"<br>  ],<br>  "userId": [<br>    1<br>  ],<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-35 | Top-level JSON array | POST /posts | '[]' | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-36 | Title length 255 ASCII characters | POST /posts | {"title": "A repeated 255 times", "body": "Boundary QA", "userId": 1}; see generator below | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {"title": "<string length=255; exact echo=True>", "body": "Boundary QA", "userId": 1, "id": 101} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. Long title exact echo: True. |
| NEG-37 | Title length 256 ASCII characters | POST /posts | {"title": "A repeated 256 times", "body": "Boundary QA", "userId": 1}; see generator below | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {"title": "<string length=256; exact echo=True>", "body": "Boundary QA", "userId": 1, "id": 101} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. Long title exact echo: True. |
| NEG-38 | Title length 10000 ASCII characters | POST /posts | {"title": "A repeated 10000 times", "body": "Boundary QA", "userId": 1}; see generator below | application/json | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {"title": "<string length=10000; exact echo=True>", "body": "Boundary QA", "userId": 1, "id": 101} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. Long title exact echo: True. |
| NEG-39 | Method outside documented API: TRACE | TRACE /posts/1 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 405 | Response Content-Type: text/html; body: <html><br><head><title>405 Not Allowed</title></head><br><body><br><center><h1>405 Not Allowed</h1></center><br><hr><center>cloudflare</center><br></body><br></html><br> | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. Method is outside README; 404/405/501 not prescribed. Gateway rejection may prevent reaching API. |
| NEG-40 | Method outside documented API: PROPFIND | PROPFIND /posts/1 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 404 | Response Content-Type: application/json; charset=utf-8; body: {} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. Method is outside README; 404/405/501 not prescribed. Gateway rejection may prevent reaching API. |
| NEG-41 | Method outside documented API: BREW | BREW /posts/1 | (no body) | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 400 | Response Content-Type: (absent); body: (empty) | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. Method is outside README; 404/405/501 not prescribed. Gateway rejection may prevent reaching API. |
| NEG-42 | Content-Type text/plain | POST /posts | '{"title":"QA content type","body":"Synthetic","userId":1}' | text/plain | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. No requirement for 415 is documented; ignored payload is an observation. |
| NEG-43 | Content-Type application/xml | POST /posts | '{"title":"QA content type","body":"Synthetic","userId":1}' | application/xml | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. No requirement for 415 is documented; ignored payload is an observation. |
| NEG-44 | Content-Type invalid | POST /posts | '{"title":"QA content type","body":"Synthetic","userId":1}' | invalid | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. No requirement for 415 is documented; ignored payload is an observation. |
| NEG-45 | Content-Type application/json; charset=invalid-charset | POST /posts | '{"title":"QA content type","body":"Synthetic","userId":1}' | application/json; charset=invalid-charset | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 500 | Response Content-Type: text/html; charset=utf-8; body: UnsupportedMediaTypeError: unsupported charset "INVALID-CHARSET"<br>    at jsonParser (/app/node_modules/body-parser/lib/types/json.js:127:12)<br>    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)<br>    at trim_prefix (/app/node_modules/express/l ... [truncated; 814 UTF-8 bytes total] | Fail | Robustness failure; upstream attribution/root cause requires review. Do not infer undocumented validation requirements. No requirement for 415 is documented; ignored payload is an observation. Reproduced once more with HTTP 500; see findings. |
| NEG-46 | Content-Type (absent) | POST /posts | '{"title":"QA content type","body":"Synthetic","userId":1}' | (absent) | Characterization: observe acceptance/rejection; any 2xx–4xx response, no 5xx. No mandatory-field, type, size, media-type, or method-specific rejection rule assumed. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. No requirement for 415 is documented; ignored payload is an observation. |
| NEG-47 | Malformed JSON regression | POST /posts | '{"title":' | application/json | HTTP client error (4xx) for malformed JSON; controlled error without internal stack trace (HTTP robustness expectation, not a README business rule). | 500 | Response Content-Type: text/html; charset=utf-8; body: SyntaxError: Unexpected end of JSON input<br>    at JSON.parse (<anonymous>)<br>    at parse (/app/node_modules/body-parser/lib/types/json.js:89:19)<br>    at /app/node_modules/body-parser/lib/read.js:121:18<br>    at invokeCallback (/app/node_modules/body-parser/node_modules/raw-body/index. ... [truncated; 663 UTF-8 bytes total] | Fail | Robustness failure; upstream attribution/root cause requires review. Do not infer undocumented validation requirements. Reproduced once more with HTTP 500; see findings. |
| NEG-48 | Valid POST control | POST /posts | '{"title":"QA valid control","body":"Synthetic","userId":1}' | application/json | 201; JSON echoes supplied fields and integer id; simulated creation. | 201 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "title": "QA valid control",<br>  "body": "Synthetic",<br>  "userId": 1,<br>  "id": 101<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |
| NEG-49 | Resource remains unchanged | GET /posts/1 | (no body) | (absent) | 200; JSON post id=1; application/json Content-Type; before/after bodies equal. | 200 | Response Content-Type: application/json; charset=utf-8; body: {<br>  "userId": 1,<br>  "id": 1,<br>  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",<br>  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"<br>} | Pass | Pass means the stated robustness/characterization check passed, not business validation compliance. |

## Reproduction instructions

Use the exact method, path, Content-Type and body in the table. Encoded ID characters are intentional: `%40%23%24` represents `@#$`, `%20` a space, and `%2F` a slash. Use `--path-as-is` and do not use `--fail`, so HTTP error bodies are retained. For requests without bodies, omit the header and data arguments. To send no Content-Type, use `--header 'Content-Type:'`; curl otherwise adds a default for data requests.

```sh
curl --silent --show-error --path-as-is --max-time 20 \
  --request POST 'https://jsonplaceholder.typicode.com/posts' \
  --header 'Content-Type: application/json' \
  --data-binary '{}' --dump-header /tmp/negative.headers \
  --output /tmp/negative.body --write-out '%{http_code}\n'
```

Long-string payload generator (Python standard library only; choose N=255, 256 or 10000), then send the generated file with curl `--data-binary @/tmp/negative-long.json`:

```python
import json
N = 10000
with open('/tmp/negative-long.json', 'w') as f:
    json.dump({'title': 'A' * N, 'body': 'Boundary QA', 'userId': 1}, f)
```

## Response evidence

The following evidence is embedded in this tracked report so it remains available in a pull request. Header lines below have trailing whitespace removed for Markdown; full unabridged request bodies, raw headers and response bodies are additionally retained in `/workspace/scratch/negative-api-evidence/` with `run.json`; that scratch directory is not part of the repository. Body previews below are explicitly shortened when indicated. SHA-256 values identify complete response bodies, not previews. For each 500 response the complete error body is embedded. No secrets or authentication were used.

### NEG-01

GET `/posts/1`; HTTP 200; curl exit 0. Body SHA-256: `965636bd900078aa86a714aea4de146af6d396205d5100636f1bdd2454f73420`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:05 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:06 GMT
content-type: application/json; charset=utf-8
content-length: 292
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"124-yiKdLzqO5gfBrJFrcdJ8Yq0LGnU"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1790812826
age: 11600
accept-ranges: bytes
cf-cache-status: HIT
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=ec%2FedoRWChe4UN7wuxnzy8tZqNjfcfvyxTLaqgj6dsOwQdmmaBbQ7u16TszyLsfBux%2FG5IJ6x7iDPqzMjpffo7ZsgDsdexTPCNDzGppcRTq1k4EMUlLefFRTcqUkWE9Neacgd6tFKPWwRMaiBq%2Fz"}]}
server: envoy
cf-ray: a4e63be70b2115ed-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 52

{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```

### NEG-02

GET `/posts/0`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:06 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1792637135
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
age: 13102
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=AlfvnCcvl%2BWfJEh2%2FO%2BI3hJzFugtqAcuBZRddecj7e2tvhH713GnFF6X1JcTrggXV%2BsBYM1tOJVpLwoqVcZWxgCRWuEXE5M%2FS7%2FsQa0Szbo1inNYF%2FwZMZYzBU%2FsFk6dDUqA1ihAZnZB%2Fx38%2F05C"}]}
server: envoy
cf-ray: a4e63be7de82e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 36

{}
```

### NEG-03

GET `/posts/-1`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:06 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599995
x-ratelimit-reset: 1792640735
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
age: 8751
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=sO8lMwBJQXcjwFoI32lKgSSpkLtXnyXA%2Fq4%2Fno5z57QtyjqqWCjg0u8oyY8ICmLxBm7QBmT%2FS9dpt9yxY%2BDq82WIJEv%2Fp4JRDFPbvHJQg6JdlQVGlPNK7CmMIEdCM4MSa%2FfERVfS%2FOuycYVFVeY5"}]}
server: envoy
cf-ray: a4e63be8c902e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 11

{}
```

### NEG-04

GET `/posts/101`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:06 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1792629935
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
age: 19190
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=SbIKhEHVnNsVck%2BO8x4Y08Zjh%2FDeh4r5xSN9VqK0QP5ptLacEWD28ta6pMGUQY1Ltl473tVmsV2dax7GwSf1qAqm%2BWR2Ys%2FELvlKaYmMy7uJKjWCBxanxBCHxLnw319L7nd8CS5GCKaRZzsHUHtM"}]}
server: envoy
cf-ray: a4e63be9ee575527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 35

{}
```

### NEG-05

GET `/posts/999999`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:06 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1792633535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
age: 15102
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=zKDDESQjCvpIjwKoEO6uNyYd0MJ%2B2Oi%2BoqLB5Y1IyiBMMP5Dy%2FylK4HGFBuKMenV6RG5ydCkpYyb8SQhf3KhOooGcWDpCpJgK58Pjl%2FKju%2FLmGOObFueMoD%2FKmfBVf0NSALDArm4u1xPZpSwMpQS"}]}
server: envoy
cf-ray: a4e63bea7dc9e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 12

{}
```

### NEG-06

GET `/posts/9007199254740992`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:07 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1792651535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=Cvmmh0tKPZ%2FXYTqf%2FY%2BW3f30n%2FGU3oU%2Ffh2dIrWetCRZk6%2B%2FaoOunXKGyo2uSk7W%2FNjv3%2BN%2BW3vWd%2Fjbq%2BQwRx3O6a5EradDXI81p0sVZ80az4YzSyVLgus2Qa9Z%2BR4o9c5%2BkfLKEhZl7e4GIYcd"}]}
server: envoy
cf-ray: a4e63beb7894e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 97

{}
```

### NEG-07

GET `/posts/abc`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:07 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1792626335
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=4oMQNhulr9iVVFlUA%2FcGexE1fM85HvujFytSfE5bbC%2BHxr%2FzHlZLLdoKdrG%2BrZVLZHwNKFrG0e7Tk2OqHWkVA5hjKTlk6aXIdwLxDPf6%2FtNxspxW4PnrJdH6gOcsD0%2FhuNa%2Fz9dq9BkPLp%2BvhGUQ"}]}
server: envoy
cf-ray: a4e63becab865527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 92

{}
```

### NEG-08

GET `/posts/1.5`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:06 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:07 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599998
x-ratelimit-reset: 1792651535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: EXPIRED
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=huo0Pk0Pjw6PnQ212FdzzmlbMCxjqnwO0K0W9BuQOE6TP8l%2FMgaA3R7R6ama%2FZuPKNSj%2BM7osffcT8oIQLshoa%2Fc1hk8hvDmyqvN5l8Te7Zfbx7aIUOBhMg0j13AZ%2B%2BgVCxGM%2Fo%2BHO7fCN3uBUdF"}]}
server: envoy
cf-ray: a4e63bedfcf060d0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 136

{}
```

### NEG-09

GET `/posts/%40%23%24`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:07 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:07 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599997
x-ratelimit-reset: 1792651535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=L%2BQVdkodvcjAm1srPUTFe%2By3MdaWjKJT8yRzzbKbghdUsSuTnJ%2FQufKzeK8NYXyrPcm5agPtz0va%2F3hKy8do4%2FoPlJUPidi%2BVZJYrjHQ5mh8gNvgVoqONs4ctJeQFphx0%2FlHPUIc5A2QJnqRN6Z9"}]}
server: envoy
cf-ray: a4e63bef58bb7ae8-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 156

{}
```

### NEG-10

GET `/posts/%20`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:07 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:07 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599996
x-ratelimit-reset: 1792651535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: EXPIRED
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=NiBclrE0c7BWORtG%2FzDhCHb4DAbR4vc0u0Ef6ddDgcJeNkAimXjcLLYOias37%2FCM%2BKqRGuD%2FFoFYKtNpZRT2Z93fQAl3ylb2Qz%2F5NsOZRLYM6r4RHtIcOeVdRxmGF6pcsszkHyG2LhQUG3kQGv5X"}]}
server: envoy
cf-ray: a4e63bf0ef0d15f5-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 187

{}
```

### NEG-11

GET `/posts/%2F`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:07 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:08 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599995
x-ratelimit-reset: 1792651535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=yTkAB5vZn9yD9V2bSR%2F12va3SwEnLCfDdZGHwFii9imgHtxbjbZVRUNBfYyrvMESGpqa3S9huDdwbAAXJMbDw0aiCit%2FVMs9zMCeCiU4FRY5Mcp29sdOXJPnq7TjgjXKhys4gUdb0QgUx%2FGXZV2V"}]}
server: envoy
cf-ray: a4e63bf228161ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 133

{}
```

### NEG-12

GET `/posts/01`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:07 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:08 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3598987
x-ratelimit-reset: 1792626335
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=Igu9k37Zgtrp3ZjPEPz82gSMwMf6Qca9q0nyFjHIdzhOWXqVyWi0lHBQf4MuB6Ay2%2FUQaj1A%2B%2Fj229EXxKX6kwRwgxYVvwA4m5vJvCAlgzqKxLLVvbuUgElruSs0MzM7dfRhYjDtwdivYi7oreTh"}]}
server: envoy
cf-ray: a4e63bf37d30d81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 122

{}
```

### NEG-13

GET `/users/0`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:08 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:08 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599992
x-ratelimit-reset: 1792644335
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=6hssNlU5LE%2F9cdQrGJmqBihOuiA4dYzkuNl%2FICxyJGEpfQlszeWQWZYXdV%2B2OA%2BG6k%2FroBVo%2BCngvH1%2F2K0CpRnypbO66U6sGR6seodOsbBtqIHG2pb1Ag1UuOA0YpkinrWfVlHvjW1ti70XxyBd"}]}
server: envoy
cf-ray: a4e63bf46f50e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 93

{}
```

### NEG-14

GET `/users/-1`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:08 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:08 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1792629935
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=ySovhP5OfIouIPHnHPgsidWSVjpenl6X2GpOBrN9zKmKv7Hdi9vUQ5XIQ7Gr6Uldb8WhXqwHOWZ8jhWk1nt7%2FSnK7asC2lImIf%2FAdJEXt1NVRBDA01TWV84AtIx2gBmHg2BJsHW0YMDD%2BbBV7g7D"}]}
server: envoy
cf-ray: a4e63bf569d6e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 93

{}
```

### NEG-15

GET `/users/11`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:08 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:08 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599996
x-ratelimit-reset: 1792629935
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=2%2FO9MhlldXXususJSnq20H8o9Sf%2BQ2Pf97bZSeyNtLyDbAMx0iLfMVCUzk7aIeXU20kI3ROAe7daiy9N5nWFalmIeyeqYymEIkn%2B7JUWUW9YnmwVB8m%2F5%2BtNxhAvbOHjd8p7iP1T78UVIlUUlkvI"}]}
server: envoy
cf-ray: a4e63bf6592160d0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 93

{}
```

### NEG-16

GET `/users/abc`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:08 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:08 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3595592
x-ratelimit-reset: 1792622735
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=hUs6INRfl%2BLdkAnoeoYydL7b3Z%2FOO3lG4e0pqbICXxjg6Uq57im%2FqXKYLyjccuireOEmSCzRFWenIPtrhz7U7sZAvbRX6d6baBdazlPYAUSFPLkMJ6nkWkXsQF8Fktx7T20AeqQtfppSH7Nlhlk%2B"}]}
server: envoy
cf-ray: a4e63bf74b9460d0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 94

{}
```

### NEG-17

GET `/users/%40%23%24`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:08 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:09 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599994
x-ratelimit-reset: 1792651535
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=%2BDxcjqLTX6fPZWtbjmyJnB%2FPDscUOaSmP89w2G6447qu8mmXa%2BhMW349GTmyC%2BMqFHe%2B1sYLdWnNjEs1pf%2BtuXQHfUEz%2FpgGTRLchqfCx0KSk475PkyQ%2FKnGeKP6tYBQ5brQ%2F%2BnXDyjVHO5bVaal"}]}
server: envoy
cf-ray: a4e63bf84fa015ed-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 124

{}
```

### NEG-18

GET `/comments?postId=0`; HTTP 200; curl exit 0. Body SHA-256: `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:08 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:09 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-l9Fw4VUO7kr8CvBlt4zaMCqXZ0w"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599998
x-ratelimit-reset: 1790841626
accept-ranges: bytes
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: HIT
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=hQth1%2FpezIt3PB4E%2FMH5PUk4Ait3bOoM160PXeM3QONVDMme%2FEQc6AbrrZWD6H8xZ40j%2BLoIMJR1CrMuJHvXY4qCgHEAwhjoARXi7SjQE7sRdMjyrovyjycFA8wO2H52tXQ2dVH8R6wpkdBRmSJp"}]}
server: envoy
cf-ray: a4e63bf98a7460d0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 91

[]
```

### NEG-19

GET `/comments?postId=-1`; HTTP 200; curl exit 0. Body SHA-256: `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:09 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:09 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-l9Fw4VUO7kr8CvBlt4zaMCqXZ0w"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599993
x-ratelimit-reset: 1792651535
accept-ranges: bytes
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=q%2FoL03U5rhoVb%2BRVKUXrSckpkdBI%2BN3s6Wzzs45ANLt1J5rMl1Fl%2BKXzGU6o%2FMmuxSRY3wStSCSOVTCmQ4pEwzu%2F9%2FjBHuQn8z3N%2BiGUlBxxURZRwk6YURN1Hc%2Fgjh4XjMJWeVeqgdE2h9Set%2B7c"}]}
server: envoy
cf-ray: a4e63bfa89c1d81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 98

[]
```

### NEG-20

GET `/comments?postId=101`; HTTP 200; curl exit 0. Body SHA-256: `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:09 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:09 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-l9Fw4VUO7kr8CvBlt4zaMCqXZ0w"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599995
x-ratelimit-reset: 1791198026
accept-ranges: bytes
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: REVALIDATED
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=NEEkd1TOlLl%2BTQEaIhsGjlHMbOanKASjm376YdqfdeaBGdvoRhgq1%2Bc0KlF4V%2FPsTWS7ix31dY3SQ%2BSD5Iwu5nc2fK9EA9ATfyNxFNw3LwWDtzGNEViMywj9ZvqOw%2FPipSGbOyU%2FbEw7U9Gnwxnp"}]}
server: envoy
cf-ray: a4e63bfb8b951ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 104

[]
```

### NEG-21

GET `/comments?postId=abc`; HTTP 200; curl exit 0. Body SHA-256: `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:09 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:09 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-l9Fw4VUO7kr8CvBlt4zaMCqXZ0w"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599991
x-ratelimit-reset: 1792651535
accept-ranges: bytes
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=Py0newKXklVlGvpx0BVZ5fYlQzaIJpSZswWsn8nP%2FfTS4AAP0v83l9KAXIzyTH4dVRU3OxQYAmfOw9kK7uF89XqIYVws2f4qYNKUpgskzwlXWfNvBP2A5z%2BMzN3MhUsfZVbHc8Kkzlt3BLH6eWTv"}]}
server: envoy
cf-ray: a4e63bfc9d38d81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 94

[]
```

### NEG-22

GET `/comments?postId=%40%23%24`; HTTP 200; curl exit 0. Body SHA-256: `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:09 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:09 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-l9Fw4VUO7kr8CvBlt4zaMCqXZ0w"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599990
x-ratelimit-reset: 1792651535
accept-ranges: bytes
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: MISS
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=BvN%2Bp1ai%2Bp70Ijpb854%2Fv9VkwWKHGNWAgwjy5yUayda5ArzmNl5WA5yr2T1sKpNTwZGNcaX6pD%2FePJDzDO9s8Ea%2Fdv7UAU4BVYUSgdCtBy0%2F0mf42aIyd%2BA8z1DzOPnW3daLbV0TTmhzyqTSmRUb"}]}
server: envoy
cf-ray: a4e63bfd9ff5e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 105

[]
```

### NEG-23

GET `/comments?postId=`; HTTP 200; curl exit 0. Body SHA-256: `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:09 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:10 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"2-l9Fw4VUO7kr8CvBlt4zaMCqXZ0w"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3584598
x-ratelimit-reset: 1792431935
accept-ranges: bytes
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
cf-cache-status: REVALIDATED
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=7Mnk3HZYJAh5aXGBLTukGbudP4ugCR5Lw8U2GGv1WRbrNMh90%2FhrarYb8u56jDmAIjsfwafZotrO4l4UQeZnXJutLSP1aOagYG%2BqEqcfFagZum8ML5CAQzL9A%2F0%2FvF9CYUhgC3uiQ3LoBy%2F42%2B92"}]}
server: envoy
cf-ray: a4e63bfebe357ae8-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 115

[]
```

### NEG-24

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:09 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:10 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599988
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=PsIWY7wRtoEYK3AxUVY%2BN%2FUeHqfSULXEEUsXREDzlwmDD6JHp6ILhT3kb%2FAqesMia8zOw%2FudzmiyjPZjnH3B%2FhmQ5Z6f5tpTLZIAVbUIEgHBV9yxDXFr5pb8mI4SyU5PyNL1WzHYGi36WN8v0TiC"}]}
server: envoy
cf-ray: a4e63bffef1b7ae8-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 240

{
  "id": 101
}
```

### NEG-25

PUT `/posts/1`; HTTP 200; curl exit 0. Body SHA-256: `783cf32f165ff45dabdce3cfcbc8f858fe317693c527378f0e6862d4c93c2388`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:10 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:10 GMT
content-type: application/json; charset=utf-8
content-length: 13
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"d-p13Thq1mbIznE8TsFv4BmW8SRpg"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599987
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=rj0tRftj8nwsiEg4%2BEZssb6coXToCi5kbrIMn6W9vOmOv3gFZas0YiXrr5Qg6%2FY4G6Kcj%2BdYbA7oemcRsgNCneI1QIuiC2HU2RkM7xZanBaA%2Br9WFxpTSdkiPQZ8CCGwN0JA3xhKwYnp3QgOyvoy"}]}
server: envoy
cf-ray: a4e63c01cbdc1ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 230

{
  "id": 1
}
```

### NEG-26

PATCH `/posts/1`; HTTP 200; curl exit 0. Body SHA-256: `965636bd900078aa86a714aea4de146af6d396205d5100636f1bdd2454f73420`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:10 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:11 GMT
content-type: application/json; charset=utf-8
content-length: 292
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"124-yiKdLzqO5gfBrJFrcdJ8Yq0LGnU"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599986
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=NON96EeIvUxdJ7LcqDjD6N6eag2N7EHXWD%2B5TlwLM%2BIgBDsYfL7GbnWQlZ4cvW0ZUbUQ8AsG%2BrgfS5HgA16xBx%2Fa112%2F4l%2F2mJwTxdMvkOjOJ28aYMTkqjIvpif7Ry8K0ezkfCpGLkUocC6C%2FTKR"}]}
server: envoy
cf-ray: a4e63c039ce315f5-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 249

{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```

### NEG-27

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:10 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:11 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599985
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=q0RZtVcb901ZOq73Ib02Bpt%2BvToasl8w%2FbN%2F1dOQyTzInqqvUe1lUg62qnMlJRKOj%2FUuCLJGVSrBOWj%2BKJEti%2Fx1bc%2Fou%2FvVQ%2BrDpeHPHT7wfzh7qLJmi%2BXCd3y%2Bg%2Bk8z2NJMNYvIFy3Rv3xsyyy"}]}
server: envoy
cf-ray: a4e63c059ca5d81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 230

{
  "id": 101
}
```

### NEG-28

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `ebf81b4a226d22decfe30db9ddbab86952116aa8dea98dbed35e777952cd642c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:11 GMT
content-type: application/json; charset=utf-8
content-length: 66
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"42-xeOCyZPZlqnnaPcLKnHPdKTDxPg"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599984
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=iyodGyy3CZqNJFjpB1rDo8YW1kB%2FcEoOiWrlI7hdR%2Ba1AamiA1%2B%2Bp%2FmShAjrtHWW76c8s80uhGMd7AQEOZogUqEKaRVyyEco1r4njU%2Frc90%2BIWCIbScluYE3xdVdno1fwrp6T%2FF1DajoQqRW9Bfu"}]}
server: envoy
cf-ray: a4e63c07694d5527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 88

{
  "title": null,
  "body": null,
  "userId": null,
  "id": 101
}
```

### NEG-29

POST `/posts`; HTTP 500; curl exit 0. Body SHA-256: `1c03ee281e81b7b3dff7594a85feb536353ce711d4569cb54c7505aa26c68846`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 500
date: Thu, 22 Oct 2026 05:52:11 GMT
content-type: text/html; charset=utf-8
content-length: 766
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"2fe-IE65twLFaFJi9dhOLFzFeGJCMw4"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599983
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=aarwbZOJrhZqf0md9iq0PpPqn1ZXb2NbFErx6cyC%2BXLecDWQ0LZTpkSobIr31alsw58900v2kM482VGcjdQ%2FKmdeklDauxUUeZ3QPs%2BiJOEvTqIkPE95yau7iDF%2BN1E3wQE4YNWsuEtS57yfFqIy"}]}
server: envoy
cf-ray: a4e63c085b585527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 85

SyntaxError: Unexpected token 'n', "#" is not valid JSON
    at JSON.parse (<anonymous>)
    at createStrictSyntaxError (/app/node_modules/body-parser/lib/types/json.js:158:10)
    at parse (/app/node_modules/body-parser/lib/types/json.js:83:15)
    at /app/node_modules/body-parser/lib/read.js:121:18
    at invokeCallback (/app/node_modules/body-parser/node_modules/raw-body/index.js:224:16)
    at done (/app/node_modules/body-parser/node_modules/raw-body/index.js:213:7)
    at IncomingMessage.onEnd (/app/node_modules/body-parser/node_modules/raw-body/index.js:273:7)
    at IncomingMessage.emit (node:events:514:28)
    at endReadableNT (node:internal/streams/readable:1783:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:90:21)
```

### NEG-30

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `630132031bde2b0061a38f42a2b9358e59d7f2861e1887911724ce0789e2e697`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:11 GMT
content-type: application/json; charset=utf-8
content-length: 61
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"3d-rZBJvD/qHQtkxPJSkPI0QE4xZf8"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599982
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=oVaWpU9DV3mGVSPKYX5uN5P0Uk4OaQ2Wf55F9Yyq6szoW2iPW8bXoQt7PGxV51f0iAJ3NbJVEhDtNWR5O6xer9iDwDAIxbXUKTJgrgz7%2F3WKlCVVuYhR7uG0QTWi6HG%2B5gldWvzrdGeS2baIMZgx"}]}
server: envoy
cf-ray: a4e63c092d771ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 89

{
  "body": "Synthetic QA data",
  "userId": 1,
  "id": 101
}
```

### NEG-31

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `94237b5e54eb665851c3a358fb856958b5239a8f542a196f4fb733a54c64411f`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:11 GMT
content-type: application/json; charset=utf-8
content-length: 58
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"3a-WGQKCtSWdq1ER5FO2R8C2YG4qEw"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599981
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=A85tedbwAxrw%2BTaFwdPNgJn2QhuHWXeXguHAsoZqtCZLQsSdCMB%2BWrcMkli0PdUDWoTAK6qPzqF9NeE39WqCkzPQ0%2Bs9v%2BM%2BmfjInO%2B78%2BSKUclztrzlsZqE7aQi%2FuY6EhgGY2M7%2BND0MiNY6FKC"}]}
server: envoy
cf-ray: a4e63c0a1ef25527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

{
  "title": "Negative test",
  "userId": 1,
  "id": 101
}
```

### NEG-32

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `ea1c81fc8ce437a65efed03801a8091f356a5bedf1752c3a5a1c070cd7fd2117`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
content-length: 74
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"4a-Jz9rodJ+QAWJCRinbFhb8zOu+A0"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599980
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=wG4sOeRuXQk92%2FfX7jWUsVig%2FzyLT3ddG5Ry8EjwesIFilMelJzUR4pCbCO4j1fUkHUYWszFNnE7Bh59mGJ1qOWCDH7saR3TS9n20H2lTjEsXz5UMjYxob9uzTI4QqsYT6Cm9RvtitlIlkEN73RN"}]}
server: envoy
cf-ray: a4e63c0b0f1cd81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

{
  "title": "Negative test",
  "body": "Synthetic QA data",
  "id": 101
}
```

### NEG-33

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `05b83905374c2b31beed0099ee167069b072b3da4be14be9ab4ca65144432b88`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
content-length: 67
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"43-1bXsHYfMa3QHil8kNKgzt77W8ho"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599979
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=FHoVdKR%2FArxhtSQtsMMO0lsVsRxCMF%2BNxzbKrC5IuZYLJebMJmkiAB00Er9ETypgX5ly2aJyY%2BTgKJCWqNk3VnvbM0MYePpdoS23ugpLLJgSL%2B0Hl8qLS0M9RmYo1jO1b0S1jjYcqrRJeHPg%2Fnxk"}]}
server: envoy
cf-ray: a4e63c0be8ced81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 89

{
  "title": 123,
  "body": false,
  "userId": "abc",
  "id": 101
}
```

### NEG-34

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `057df7118ea81475fbfd780712f9bec450a8dcf28c21ed4a0fd92c658a3b795a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:11 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
content-length: 94
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"5e-y56gTxKbztO8XyyYB8Lc9kiAc/8"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599978
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=yFgkK4jLge7ZqPYWgVKSQjWN8uVepKHl2W5BeuMNq0SFjzr5jNiTojCxhKJqzykGDKJ9cS%2FRisNeF%2BNY%2B%2Fz4VSoRhNxJPOgOC4E30EOzFIIVq8UgbYGY1TFPBQ7v6eNmeFFmhF%2FKHz8IfjnTQpNb"}]}
server: envoy
cf-ray: a4e63c0cd8b07ae8-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 97

{
  "title": {
    "x": 1
  },
  "body": [
    "x"
  ],
  "userId": [
    1
  ],
  "id": 101
}
```

### NEG-35

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599977
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=fh2gStQ8VUvvsinit%2FLPDyWrpoFLj1cC7vigQOFfOlNVVboSmJ6gEGtwmSGa%2FHY%2F1Yt%2BqGO%2FC3IdRBKmmOJAUo5jtEDd92zXs8jB8tGaOw57P4fRDwmtQYPQ2T1BDmeNrYjSu2NMaeuW%2B3aX0NZh"}]}
server: envoy
cf-ray: a4e63c0dcbf4d81b-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

{
  "id": 101
}
```

### NEG-36

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `a0ea3acf4e4e8fa0d0497a0a2ca6f1f2d5f2ef4182001da90c5d86378ced52e4`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
content-length: 325
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"145-q3wMm76cVE0gvOQnMO6qV3TSr/4"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599976
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=RWcgRyrLNYnYEMkliwCRulYD6UXBKiXVwB6VqEYsCYmMeoP29fCjhyM2jkDkqXIoHZax1QTFOytEIcGZNi5P3K9wqUTu19ZH8MQ%2FGBY6x2A3SUS5Shjx%2BVrjk9HvMwJ3x4YCTVVYaqwNU22syBxl"}]}
server: envoy
cf-ray: a4e63c0ebd411ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

{
  "title": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "body": "Boundary QA",
  "userId": 1,
  "id": 101
}
```

### NEG-37

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `9e5fb30b9e0711c7f9fe764f6bc6761dfd62abfc71c5d94ba05ea5a3426a7f92`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
content-length: 326
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"146-f2G/aLwl2/SIXR2wiIe2LIb23v8"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599975
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=PtONs1Pi9QUJv16Ljsa1Postmo2%2B3xTjk4X%2FLzNESzRabB%2Fy3EQXEyCfRkXjoMpSbnUPLJ2bGP1Za%2FJNomuNm2w3xLdop24c%2BXFCyROhDwEBIjJkU9xRpkQ8H4qXewk4yfEBbssvFvMLL6A44BP%2F"}]}
server: envoy
cf-ray: a4e63c0fae781ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

{
  "title": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
  "body": "Boundary QA",
  "userId": 1,
  "id": 101
}
```

### NEG-38

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `b119d332f97eb62cc4ab5e89390e5f6f34024f337dd8ee7bdd06991503a18187`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:12 GMT
content-type: application/json; charset=utf-8
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=ZFKCar2qAoPGmWDYd%2FcB2bW2jAYUzuSoKvCdxd0R1evG5sLKuxnI2usxPxTI6x7ILzm6HbayszI6lkDD0dnHAhgvrNwSRL9ct4SobMXlEQVTKR15ao5BhY4DWXVnkIbyafNNGN9trqO%2BRrdbVyWT"}]}
etag: W/"2756-+GhC79NAEd0gEVw+XcMxvep6hug"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599974
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
server: envoy
cf-ray: a4e63c109b435527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 88

{"title": "<string length=10000; exact echo=True>", "body": "Boundary QA", "userId": 1, "id": 101}
```

### NEG-39

TRACE `/posts/1`; HTTP 405; curl exit 0. Body SHA-256: `27baada7b22391a1f805a9e034230416c4e65fb33d69b6b471610276ab93d3cc`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 405
server: envoy
date: Thu, 22 Oct 2026 05:52:13 GMT
content-type: text/html
content-length: 155
cf-ray: -
x-envoy-upstream-service-time: 3

<html>
<head><title>405 Not Allowed</title></head>
<body>
<center><h1>405 Not Allowed</h1></center>
<hr><center>cloudflare</center>
</body>
</html>

```

### NEG-40

PROPFIND `/posts/1`; HTTP 404; curl exit 0. Body SHA-256: `44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 404
date: Thu, 22 Oct 2026 05:52:13 GMT
content-type: application/json; charset=utf-8
content-length: 2
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"2-vyGp6PvFo4RvsFtPoIWeCReyIC8"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599973
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=Z1Qo6hkt%2BoBr0vEtnAhKV2OUhcutrnVJVcUNGJN%2BhgD5z3cVNF69TU8xM6PKy%2FhKSuIb9oWp9114111kn5DW95w0JpTUWOzkJkQvdDx4Ja1Qs9aWyP74%2Bnf1GXk1egDb9QRuYq4upfUlH%2FR%2FRvrF"}]}
server: envoy
cf-ray: a4e63c11fc847ae8-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 93

{}
```

### NEG-41

BREW `/posts/1`; HTTP 400; curl exit 0. Body SHA-256: `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:12 GMT
server: envoy

HTTP/2 400
date: Thu, 22 Oct 2026 05:52:13 GMT
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=Yu40tNEda%2FGQvQCNAVps1ER8ayE%2B35HwtO5i2Rgmanpvq1STLIdqlQJpFzgbjq6MK336iU%2FQH0Ruz5iNvE0ckcTkM744%2BkXlZrfX3I0B%2F%2FlVJoSM6e2IHvqaDxfXJ1bOvczBaomdN4dFAQmpHm8Z"}]}
server: envoy
cf-ray: a4e63c12fb291ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 86


```

### NEG-42

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:13 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599972
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=Mqt5PFJgOPzwQ1kdFnCYdcE%2ButeFXwymPogkGBwa%2FtkWNIJCMaG3WMpe%2Fq2wZVpK%2FmZnT2HgOvAO9L%2Fz34ruCop28HG78ncJaqKaqTPgEgtQrQ%2F0FdxWThA2%2FZAVStb48cfohcYjFa7Js%2BYTjQJO"}]}
server: envoy
cf-ray: a4e63c13ea815527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 86

{
  "id": 101
}
```

### NEG-43

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:13 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599971
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=5FYYfv2BmfepRPScqgNY%2FmrfmfXwxX7tBNY0AiPi%2F0EVXRvEJa%2FNsKPnyXkT6Zq8nKi9n5ikrPZxdjvmiLt%2FnaodMvYdf477RwJ%2FlFgnjvHvqUesPxQ1MZ8hgMk0LOcxz7fWbb0ytXtOYOq%2F%2FUFU"}]}
server: envoy
cf-ray: a4e63c150b6ee965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 83

{
  "id": 101
}
```

### NEG-44

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:13 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599970
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=zQM8%2BTIU%2BaiIGieXmB9g2ixJ6DhJeza3N5Vcg4AZmJOCgs%2FH0iFksQYbzbdXuWhToToHPl2bPfJ22F3U%2BiaVLTV0eEv8EXKnLIQIZjHe0kcg6Os%2FPParlA02CkhR4VWLDJlu6uY6G5nHXPE%2BOSgf"}]}
server: envoy
cf-ray: a4e63c15e9cb15f5-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 98

{
  "id": 101
}
```

### NEG-45

POST `/posts`; HTTP 500; curl exit 0. Body SHA-256: `d8ffdfce2ec0ece036c03503571c1325cff037b0bac1c7679ebb5a9860e0abd5`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 500
date: Thu, 22 Oct 2026 05:52:13 GMT
content-type: text/html; charset=utf-8
content-length: 814
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"32e-RjrF/o2S48bLrObzP1G/ZgrmFoI"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599969
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=0GxdNO3R5%2B2oLA8ExqvYFOe3bgezTh3uJcmL4j1gLbKBhAgWcjqEayIy34pQmEqTqcE7t5x1Qb3WI7o8ytK2kn2AdrotcsuTvZ4VRyIWMf8DxejgF3u97irAKfNX2xtcJCC67mJ7BRJir%2BoiRuJv"}]}
server: envoy
cf-ray: a4e63c16effa1ff0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

UnsupportedMediaTypeError: unsupported charset "INVALID-CHARSET"
    at jsonParser (/app/node_modules/body-parser/lib/types/json.js:127:12)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at trim_prefix (/app/node_modules/express/lib/router/index.js:317:13)
    at /app/node_modules/express/lib/router/index.js:284:7
    at router.process_params (/app/node_modules/express/lib/router/index.js:335:12)
    at next (/app/node_modules/express/lib/router/index.js:275:10)
    at methodOverride (/app/node_modules/method-override/index.js:79:5)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at trim_prefix (/app/node_modules/express/lib/router/index.js:317:13)
    at /app/node_modules/express/lib/router/index.js:284:7
```

### NEG-46

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `49fae3f018950cf666303d82277effd86436173dc712696cb83662d8cafaf53c`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: application/json; charset=utf-8
content-length: 15
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"f-4jjw4Y8q22Yv1PV9m28FczJgjzk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599968
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=xSVcxYRKmFQGDATh8vILLdO4PYGSaL0hZCBS%2FpogNZ6R5wRe0bn6%2BOk4YG7d4yhjF9RWUJm3Uz2YRGTNlk%2BmTU0xo4wmvGntsCsugpPVHLGTShIUC6F%2FZxY4B5ioNeaBF9Kqka%2BKVM5hcoecqcSg"}]}
server: envoy
cf-ray: a4e63c17dc5e60d0-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 87

{
  "id": 101
}
```

### NEG-47

POST `/posts`; HTTP 500; curl exit 0. Body SHA-256: `0e20b3e9743ee87f56cb122914c5350caae19218e44344aaba76cf03d41a47fc`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 500
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: text/html; charset=utf-8
content-length: 663
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"297-3uW8ncQCVaQLzZkYAYvNJHmYOKk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599967
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=GNq2u6ldGXUH01Nib8GSCsw5%2Flxw2wLBIH0jYFpyxNByl19023lz29Dfya7G40ib%2ByN55orAURz%2FNhIeq3j1lU0hSc4BEiqgryiv5E11asjUbQUzdvP%2BVrFJy0ICfEWn8Oo1AAfgzTBysuxrWTTo"}]}
server: envoy
cf-ray: a4e63c18bf8115ed-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 94

SyntaxError: Unexpected end of JSON input
    at JSON.parse (<anonymous>)
    at parse (/app/node_modules/body-parser/lib/types/json.js:89:19)
    at /app/node_modules/body-parser/lib/read.js:121:18
    at invokeCallback (/app/node_modules/body-parser/node_modules/raw-body/index.js:224:16)
    at done (/app/node_modules/body-parser/node_modules/raw-body/index.js:213:7)
    at IncomingMessage.onEnd (/app/node_modules/body-parser/node_modules/raw-body/index.js:273:7)
    at IncomingMessage.emit (node:events:514:28)
    at endReadableNT (node:internal/streams/readable:1783:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:90:21)
```

### NEG-48

POST `/posts`; HTTP 201; curl exit 0. Body SHA-256: `eabe059ec792e99539492a9c2c5fa4599053eac5197253b79b1ee9da510e87ab`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:13 GMT
server: envoy

HTTP/2 201
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: application/json; charset=utf-8
content-length: 84
location: https://jsonplaceholder.typicode.com/posts/101
access-control-allow-credentials: true
access-control-expose-headers: Location
cache-control: no-cache
etag: W/"54-S2RzMOvFUq1HD/vI+iKSHATxU2s"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599966
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=iybmYG9gHxJnrXQxKRxLIkd67dJ%2FcHhNEkwd4IsU9xsZl7NL30yCxEWI6Yo9GmBBBP4VS9b8VBAkBPjURmVTzJEjYXiHeajQBEabuhI7Wo4CHqVZif0vhSxFXT%2FReV%2FXOIz4q4PdthOBPacr7jaD"}]}
server: envoy
cf-ray: a4e63c19b86315ed-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 84

{
  "title": "QA valid control",
  "body": "Synthetic",
  "userId": 1,
  "id": 101
}
```

### NEG-49

GET `/posts/1`; HTTP 200; curl exit 0. Body SHA-256: `965636bd900078aa86a714aea4de146af6d396205d5100636f1bdd2454f73420`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:14 GMT
server: envoy

HTTP/2 200
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: application/json; charset=utf-8
content-length: 292
access-control-allow-credentials: true
cache-control: max-age=43200
etag: W/"124-yiKdLzqO5gfBrJFrcdJ8Yq0LGnU"
expires: -1
pragma: no-cache
vary: Origin, Accept-Encoding
x-content-type-options: nosniff
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599999
x-ratelimit-reset: 1790812826
age: 11608
accept-ranges: bytes
cf-cache-status: HIT
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=uukpwuZzbLmcCk6IB8RepPA7HIUFXN0mvl28ddJTS7zBviIB5dqGzx30BWEy8DFKNm1xnZDEEydrNXU0iorsSdShETArhSANI8Qv5HrIhnW%2FkKAT13yWvHE1NyOnVSCQWbb88eLBf3qBIFbIrGY%2B"}]}
server: envoy
cf-ray: a4e63c1aaf345527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 9

{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```

### NEG-29 reproduction

POST `/posts`; HTTP 500; curl exit 0. Body SHA-256: `1c03ee281e81b7b3dff7594a85feb536353ce711d4569cb54c7505aa26c68846`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:14 GMT
server: envoy

HTTP/2 500
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: text/html; charset=utf-8
content-length: 766
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"2fe-IE65twLFaFJi9dhOLFzFeGJCMw4"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599965
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=WmMhEmgrLx2bZv7QvcOIbEB8CnySajHepYSKzcDXEQJC4hyFFBeszCQ17xIRBpWi6dkkN0H5XJcKIugLaV%2FWTN3ZVm7VhTSL7yY4AzStGhgQaUVGzErA9piVBEom1of1Mvg48etVSS43rE5cgeps"}]}
server: envoy
cf-ray: a4e63c1b28075527-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 85

SyntaxError: Unexpected token 'n', "#" is not valid JSON
    at JSON.parse (<anonymous>)
    at createStrictSyntaxError (/app/node_modules/body-parser/lib/types/json.js:158:10)
    at parse (/app/node_modules/body-parser/lib/types/json.js:83:15)
    at /app/node_modules/body-parser/lib/read.js:121:18
    at invokeCallback (/app/node_modules/body-parser/node_modules/raw-body/index.js:224:16)
    at done (/app/node_modules/body-parser/node_modules/raw-body/index.js:213:7)
    at IncomingMessage.onEnd (/app/node_modules/body-parser/node_modules/raw-body/index.js:273:7)
    at IncomingMessage.emit (node:events:514:28)
    at endReadableNT (node:internal/streams/readable:1783:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:90:21)
```

### NEG-45 reproduction

POST `/posts`; HTTP 500; curl exit 0. Body SHA-256: `d8ffdfce2ec0ece036c03503571c1325cff037b0bac1c7679ebb5a9860e0abd5`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:14 GMT
server: envoy

HTTP/2 500
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: text/html; charset=utf-8
content-length: 814
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"32e-RjrF/o2S48bLrObzP1G/ZgrmFoI"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599964
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=wwIYo%2BB%2FHsMMSEKMJFnUPJXe%2FZ5B8mFtMklf54bxpaMVnesrlsT%2FpSFwElX6WFj%2FFDeZH8TkkaTh%2BLgpLLeuzruC3tU7E87rhYQxx%2BFAAwEJDjoIbgLranIr%2BjN9aCLEot3pfnAgFXnYHBmq%2Fcwf"}]}
server: envoy
cf-ray: a4e63c1c2efa15f5-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 90

UnsupportedMediaTypeError: unsupported charset "INVALID-CHARSET"
    at jsonParser (/app/node_modules/body-parser/lib/types/json.js:127:12)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at trim_prefix (/app/node_modules/express/lib/router/index.js:317:13)
    at /app/node_modules/express/lib/router/index.js:284:7
    at router.process_params (/app/node_modules/express/lib/router/index.js:335:12)
    at next (/app/node_modules/express/lib/router/index.js:275:10)
    at methodOverride (/app/node_modules/method-override/index.js:79:5)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at trim_prefix (/app/node_modules/express/lib/router/index.js:317:13)
    at /app/node_modules/express/lib/router/index.js:284:7
```

### NEG-47 reproduction

POST `/posts`; HTTP 500; curl exit 0. Body SHA-256: `0e20b3e9743ee87f56cb122914c5350caae19218e44344aaba76cf03d41a47fc`.

```http
HTTP/1.1 200 OK
date: Thu, 22 Oct 2026 05:52:14 GMT
server: envoy

HTTP/2 500
date: Thu, 22 Oct 2026 05:52:14 GMT
content-type: text/html; charset=utf-8
content-length: 663
access-control-allow-credentials: true
cache-control: no-cache
etag: W/"297-3uW8ncQCVaQLzZkYAYvNJHmYOKk"
expires: -1
pragma: no-cache
vary: Origin, X-HTTP-Method-Override, Accept-Encoding
x-powered-by: Express
x-ratelimit-limit: 3600000
x-ratelimit-remaining: 3599963
x-ratelimit-reset: 1792651535
cf-cache-status: DYNAMIC
nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
report-to: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=lTtRZj8QzenCG0Z6%2FsYxQmAbQm9ugtxWYNtFl9M5zwt8Wo49MnGoAZSu3mBnNgio7LN2ywstBO3oF7z77FLu%2BjaAH9ze%2BT0HLAqd34AfD7yTHRHbcy8BLYxJbTH9My2IdVNgzWT5bQNYRRboiftN"}]}
server: envoy
cf-ray: a4e63c1d28b5e965-SJC
alt-svc: h3=":443"; ma=86400
x-envoy-upstream-service-time: 82

SyntaxError: Unexpected end of JSON input
    at JSON.parse (<anonymous>)
    at parse (/app/node_modules/body-parser/lib/types/json.js:89:19)
    at /app/node_modules/body-parser/lib/read.js:121:18
    at invokeCallback (/app/node_modules/body-parser/node_modules/raw-body/index.js:224:16)
    at done (/app/node_modules/body-parser/node_modules/raw-body/index.js:213:7)
    at IncomingMessage.onEnd (/app/node_modules/body-parser/node_modules/raw-body/index.js:273:7)
    at IncomingMessage.emit (node:events:514:28)
    at endReadableNT (node:internal/streams/readable:1783:12)
    at process.processTicksAndRejections (node:internal/process/task_queues:90:21)
```
