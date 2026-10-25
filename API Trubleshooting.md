
|   |   |   |   |   |
|---|---|---|---|---|
|Rank|Topic|What it is (one line)|Why it matters for this job|Interview frequency|
|1|API troubleshooting|Finding why a request between two systems fails|Most customer tickets are "the integration is broken"|Very high|
|2|JavaScript core|The language that runs web pages and Node.js servers|You read customer code, console errors and scripts|Very high|
|3|Webhooks|A server calling _your_ URL when an event happens|"We are not receiving events" is a classic ticket|High|
|4|Browser DevTools + support skills|Console, Network tab, ticket handling|Shows _how_ you debug, not just what you know|High|
|5|HTML|The structure of a web page|Broken widgets, forms, embeds|Medium|
|6|CSS|The look and layout of a page|"Button hidden / layout broken" tickets|Medium|
|7|Browser extensions|Small programs that add features to Chrome/Firefox|Listed in the JD; extensions often break pages|Medium|
|8|Patterns/algorithms in support|Retry with backoff, rate limiting, debounce, bisecting|Shows engineering depth|Low–medium|

**How to study:** read each section's _What & why_, then _How it works_, then type out the code yourself. End with the question bank in the last section.

`Latency `: is the time that the server takes to respond for the client request

`response time` : total time required to get the the response from server starting from request

## 1. API troubleshooting

**What & why:** An API (Application Programming Interface) is a set of rules that lets one program ask another program for data or actions. _API troubleshooting_ means finding out **why a request did not give the expected result** — wrong data, an error code, slowness, or no response at all. This is the single most common task in this job.

### 1.1 Key definitions

- **Client** — the program that sends the request (browser, mobile app, Postman, a script).
- **Server** — the program that receives it and sends back a response.
- **Endpoint** — a specific URL that does one job, e.g. `https://api.shop.com/v1/orders`.
- **REST** — the most common API style: resources are URLs, actions are HTTP methods, data is usually JSON.
- **JSON** — text format for data: `{"id": 42, "status": "paid"}`.
- **Stateless** — every request must carry everything needed (including auth); the server does not remember the previous request.
- **Idempotent** — sending the same request twice has the same effect as once (GET, PUT, DELETE are; POST usually is not).

### 1.2 Anatomy of an HTTP request and response

Request = **method + URL + headers + body**. Response = **status code + headers + body**.

```http
POST /v1/orders HTTP/1.1
Host: api.shop.com
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json

{"item": "book", "qty": 2}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json

{"id": 981, "item": "book", "qty": 2}
```

|   |   |   |   |
|---|---|---|---|
|Method|Meaning|Has body?|Idempotent?|
|GET|Read data|No|Yes|
|POST|Create / trigger an action|Yes|No|
|PUT|Replace a whole resource|Yes|Yes|
|PATCH|Update part of a resource|Yes|Not guaranteed|
|DELETE|Remove a resource|Usually no|Yes|
|OPTIONS|Ask what is allowed (used by CORS preflight)|No|Yes|

### 1.3 Status codes — memorise these

The first digit tells you **whose fault it probably is**: 2xx = success, 3xx = redirect, 4xx = the client sent something wrong, 5xx = the server failed.

|           |                        |                                                                                                                  |                                                        |
| --------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Code      | Name                   | Typical cause                                                                                                    | What you check first                                   |
| 200       | OK                     | Success                                                                                                          | Is the _body_ what was expected?                       |
| 201       | Created                | Resource created                                                                                                 | Returned id                                            |
| 204       | No Content             | Success, empty body                                                                                              | Client should not try to parse JSON                    |
| 301 / 302 | Redirect               | URL moved, http → https                                                                                          | POST may become GET after redirect; use final URL      |
| 304       | Not Modified           | Cached copy is still valid                                                                                       | Caching headers (ETag)                                 |
| 400       | Bad Request            | Malformed JSON, missing/invalid field                                                                            | Request body, Content-Type, validation message         |
| 401       | Unauthorized           | Missing, wrong or expired credentials                                                                            | API key / token present? expired? correct environment? |
| 403       | Forbidden              | Authenticated but not allowed                                                                                    | Scopes / permissions / plan / IP allow-list            |
| 404       | Not Found              | Wrong URL, wrong id, wrong API version                                                                           | Typos, `/v1` vs `/v2`, resource deleted                |
| 405       | Method Not Allowed     | GET used where POST is needed                                                                                    | Method                                                 |
| 408       | Request Timeout        | Client too slow to send                                                                                          | Network, payload size                                  |
| 409       | Conflict               | Duplicate or state conflict                                                                                      | Already exists? concurrent update?                     |
| 413       | Payload Too Large      | Body over limit                                                                                                  | File size limits                                       |
| 415       | Unsupported Media Type | Wrong `Content-Type`                                                                                             | Send `application/json`                                |
| 422       | Unprocessable Entity   | Valid JSON, but values break rules                                                                               | Field-level error message                              |
| 429       | Too Many Requests      | Rate limit hit                                                                                                   | `Retry-After`, rate-limit headers, add backoff         |
| 500       | Internal Server Error  | Bug on the server                                                                                                | Server logs, request id; reproduce                     |
| 502       | Bad Gateway            | I am the gateway. I talked to the backend, but the backend gave me a bad answer                                  |                                                        |
| 503       | Service Unavailable    | Overloaded or in maintenance. temporarily the server is not available, may be due to the overload on the server. | Status page, retry later                               |
| 504       | Gateway Timeout        | Upstream took too long                                                                                           | Slow query, big payload, timeouts                      |

### 1.4 Important headers

- `Content-Type` — format of the body you _send_ (`application/json`, `multipart/form-data`).
- `Accept` — format you _want back_.
- `Authorization` — credentials, e.g. `Bearer <token>` or `Basic base64(user:pass)`.
- `X-Request-Id` / `Request-Id` — a unique id per request; quote it to engineering to find the log line.
- `Retry-After`, `X-RateLimit-Remaining` — rate limiting info.
- `Cache-Control`, `ETag` — caching.
- `Access-Control-Allow-Origin` — CORS (below).

### 1.5 Authentication types

|                    |                                                                                                                                                                                                                                                                                    |                                                                    |     |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | --- |
| Type               | How it is sent                                                                                                                                                                                                                                                                     | Common failure                                                     |     |
| API key            | Header (`x-api-key: abc`) or query string. provide the user with a api key , which will be sent over the header or in query parameter. when the backend processes it , it checks for that key in the database. if it exist, then continues.                                        | Test key used in production, key revoked, extra spaces copied      |     |
| Basic auth         | `Authorization: Basic base64(user:pass)` takes username and password, hashes it(not encryption) using base64.encode() , but if someone got this token, then they can easily decode using base64.decode(). this is not encryption.                                                  | Not base64-encoded, wrong password                                 |     |
| Bearer token / JWT | `Authorization: Bearer <token>` uses username and password, when the firt time user logins using that, it will provide a token, from the next time he can directly acess the resource using that token without login. Bearer means whoever posses this token, allow them to access | Token expired, missing the word `Bearer`, wrong audience           |     |
| OAuth 2.0          | Get an access token from an auth server, then use as Bearer                                                                                                                                                                                                                        | Access token expired → must use refresh token; missing scope → 403 |     |

**OAuth 2.0 answers:**

> "How do I obtain an access token, and what permissions should it have?"

**Bearer authentication answers:**

> "How do I send that access token to the API?", it is  is an authentication scheme used to send an access token to an API.

**JWT (JSON Web Token)** has three base64 parts: `header.payload.signature`. You can decode the payload (e.g. on jwt.io) to see `exp` (expiry time) and `scope` — a fast way to diagnose 401/403.

### 1.6 CORS — the most confusing browser error

**What:** CORS (Cross-Origin Resource Sharing) is a _browser_ security rule. A page on `https://myapp.com` cannot read a response from `https://api.other.com` unless that server replies with `Access-Control-Allow-Origin: https://myapp.com` (or `*`).

**How it works:** for "non-simple" requests (custom headers, JSON body, PUT/DELETE) the browser first sends an **OPTIONS preflight** request. If the server's reply lacks the right `Access-Control-Allow-*` headers, the browser blocks the real request.

**Key facts to say in the interview:**

- CORS errors happen **only in browsers** — the same request works in Postman or curl. That is the tell-tale sign.
- The fix is on the **server** (add allowed origins), not in the client JavaScript.
- Never "fix" CORS by putting a secret API key in front-end code.

## How CORS works??

you specify the allowed origins/url's in CORS, when a browser sends the request to the backend. it always reaches to the backend api. then the api also returns the response. but after the response is sent to the browser, browser check the  the allow origin in OCRS that is i allowed to read this response. it is only allowed if the request url is listed under allow origin. else it volute rely blocks and throws CORS error

`thats why even if the status of the request  in the logs show 200ok, but it throws CORS blocked excception sometimes`

The **CORS mechanism is enforced by the browser**, not by your FastAPI server itself.
### 1.7 The troubleshooting method (step by step)

1. **Reproduce** — To reproduce a bug = follow the same steps/conditions that cause the bug so you can observe it yourself. get the exact request: method, URL, headers, body, time, environment (test/live), request id.

2.  **Read the error** : status code + response body. Most APIs explain the problem in the body.

3. **Check logs**  : client console, API dashboard logs, server logs by request id.

4. **Isolate**  : try the same call in Postman/curl. Works there but not in the app → the problem is in the app (CORS, code, environment variables). Fails everywhere → request or server.

5. **Check the usual suspects** in order: URL & version → method → auth → headers (`Content-Type`) → body/JSON validity → permissions → rate limits → server status.

6. **Compare with a working request** — change one thing at a time.

7. **Fix, verify, document** — confirm with the customer, write what caused it.
### 1.8 API Development and Testing Tools

- **Browser DevTools → Network tab** — every request, its headers, payload, response, timing.
- **Postman / Insomnia** — build and replay requests by hand.
- **EchoAPI** : another tool for development and testing
- **curl** — command-line requests; easy to share in tickets.
- **jq** — read JSON in the terminal.
- **API provider dashboard logs**, status page.

```bash
# -i shows response headers, -v shows the whole conversation (TLS, redirects)
curl -i -X POST https://api.shop.com/v1/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"item":"book","qty":2}'
```
### 1.9 Code: calling an API safely in JavaScript

```js
async function createOrder(order) {
  const res = await fetch("https://api.shop.com/v1/orders", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${process.env.API_TOKEN}`
    },
    body: JSON.stringify(order) // forgetting stringify is a classic 400
  });

  // fetch does NOT throw on 4xx/5xx — you must check res.ok yourself
  if (!res.ok) {
    const errorBody = await res.text(); // text(): the error may not be JSON
    throw new Error(`HTTP ${res.status}: ${errorBody} (request id: ${res.headers.get("x-request-id")})`);
  }
  return res.json();
}

createOrder({ item: "book", qty: 2 })
  .then(data => console.log("Created", data))
  .catch(err => console.error(err.message));
```

### Fetch() :
- if the fetch successfully receives the http response , then it returns status code, and it will be catched under then() block in promise. even though the status code is 4XX or 5XX
- it only handled under error() block if it does not receives a valid response at all.
- if fetch successfully returns a block, then it returns response.ok else it will not contain any res.ok() block. so you can check for this to know whether the reponse is under 200 or 300 range or not.

### 1.10 Common scenarios and answers

|                                    |                                                                 |                                               |
| ---------------------------------- | --------------------------------------------------------------- | --------------------------------------------- |
| Symptom                            | Most likely cause                                               | Fix                                           |
| Works in Postman, fails in browser | CORS                                                            | Server adds allowed origin                    |
| Worked yesterday, 401 today        | Token expired / key rotated                                     | Refresh token, update key                     |
| 400 with "invalid JSON"            | Body not stringified, trailing comma, wrong quotes              | `JSON.stringify`, validate JSON               |
| 415                                | Missing `Content-Type: application/json`                        | Add header                                    |
| 404 on a correct-looking URL       | Wrong version, wrong base URL (sandbox vs live), trailing slash | Compare with docs                             |
| Intermittent 429                   | Burst of requests                                               | Backoff + queue (section 8)                   |
| Intermittent 502/504               | Server slow or overloaded                                       | Retry with backoff, escalate with request ids |
| `SSL certificate problem`          | Expired cert, corporate proxy, wrong system clock               | Check cert, clock                             |
| Data missing in response           | Pagination — only first page returned                           | Follow `next` cursor / `page`                 |



## 3. Webhooks

**What & why:** A webhook is an **HTTP POST that a service sends to your URL automatically when an event happens** (payment succeeded, form submitted, order shipped). It is "reverse API": instead of you asking every minute _"anything new?"_ (polling), the service tells you the moment it happens. Used by Stripe, Razorpay, GitHub, Shopify, Slack, Zapier.

|   |   |   |
|---|---|---|
||Polling (normal API)|Webhook|
|Who starts the call|Your app|The provider|
|Timing|Every N seconds/minutes|Instantly, on event|
|Wasted calls|Many ("nothing new")|None|
|You need|API key|A public HTTPS URL (endpoint)|

### 3.1 How it works (step by step)

1. You **register** an endpoint URL in the provider's dashboard (e.g. `https://myapp.com/webhooks/payments`) and choose events.
2. The provider gives you a **signing secret**.
3. An event happens → the provider sends a **POST** with a JSON body (event type, id, data) and a **signature header**.
4. Your server **verifies the signature**, quickly returns **2xx**, then processes the event (often in a background queue).
5. If your server returns non-2xx or times out, the provider **retries** (usually with exponential backoff, for hours or days) and may disable the endpoint after many failures.

[embed: node/90e8a943-78dc]

Any non-2xx reply or timeout sends the event back to the provider's retry queue, which is why duplicates happen and idempotency matters.

Example payload:

```json
{
  "id": "evt_1Nq2",
  "type": "payment.succeeded",
  "created": 1759912345,
  "data": { "payment_id": "pay_88", "amount": 49900, "currency": "INR" }
}
```

### 3.2 Key concepts

- **Signature verification (HMAC)** — the provider computes `HMAC-SHA256(secret, raw_body)` and sends it in a header. You compute the same thing; if they match, the request is genuine and unchanged. Without this, anyone could POST fake "payment succeeded" events.
- **Raw body** — the signature is computed on the _exact bytes_ received. If your framework parses JSON first and you re-stringify it, spacing changes and the signature fails. This is the #1 webhook bug.
- **Timestamp / replay protection** — reject events older than ~5 minutes so a captured request can't be replayed.
- **Idempotency** : performing the operation multiple times produces the same result, it is called idempotency
- **Respond fast** — return 200 within a few seconds; do heavy work later. Slow handlers cause timeouts → retries → duplicates.
- **Ordering** — events can arrive out of order; use timestamps or re-fetch the latest state from the API.

### 3.4 Testing webhooks locally

- **ngrok** / **cloudflared tunnel** — gives your `localhost:3000` a public HTTPS URL.
- **webhook.site** / **RequestBin** — a throw-away URL that shows exactly what the provider sends.
- Provider CLIs (e.g. `stripe listen`, `stripe trigger payment_intent.succeeded`) and dashboard "Resend" buttons.

### 3.5 Troubleshooting: "We are not receiving webhooks"

|   |   |
|---|---|
|Check|What to look for|
|Provider's delivery log|Was it sent? What status code / error did your server return?|
|URL|Exact URL, https, no typo, correct path, endpoint enabled, right events subscribed|
|Environment|Test-mode events go to test endpoints; live to live|
|Reachability|`localhost` is not public; firewall, IP allow-list, expired SSL certificate|
|Response code|3xx redirects often count as failures; 4xx/5xx trigger retries; a crash = 500|
|Timeout|Handler doing slow work before responding|
|Signature failing|Wrong secret (test vs live), body parsed before verifying, wrong header name|
|Duplicates|Missing idempotency check; handler was slow so provider retried|
|Endpoint disabled|Too many failures → provider auto-disabled it; fix and re-enable, replay events|

**Polling vs webhooks interview answer:** use webhooks for real-time events and efficiency; keep a periodic **reconciliation** job (poll the API) as a safety net for missed events.


## 8. Algorithms and patterns used in support work

**What & why:** Support roles don't ask heavy DSA, but they _do_ ask "how would you handle rate limits / failures / duplicates?". These patterns are the "algorithms" behind API and webhook reliability.

### 8.1 Retry with exponential backoff + jitter

**Problem:** a call fails temporarily (429, 502, 503, 504, network error). Retrying instantly makes overload worse.

**Algorithm:** wait 1 s, then 2 s, 4 s, 8 s... (doubling) plus a small random delay (_jitter_) so thousands of clients don't retry at the same moment. Stop after N attempts. **Never retry 400/401/403/404** — they will fail again.

```js
async function fetchWithRetry(url, options = {}, maxRetries = 4) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const res = await fetch(url, options);
      const retryable = res.status === 429 || res.status >= 500;
      if (!retryable) return res;                       // success or a non-retryable error
      if (attempt === maxRetries) return res;

      const retryAfter = Number(res.headers.get("Retry-After")); // seconds, if server says
      const wait = retryAfter ? retryAfter * 1000 : 2 ** attempt * 1000 + Math.random() * 500;
      await new Promise(r => setTimeout(r, wait));
    } catch (networkErr) {
      if (attempt === maxRetries) throw networkErr;
      await new Promise(r => setTimeout(r, 2 ** attempt * 1000));
    }
  }
}
```

### 8.2 Rate limiting (token bucket)

**What:** servers limit requests per time (e.g. 100/minute) and return 429 when exceeded.

**Token bucket algorithm:** Keeps a fixed bucket at the backend, whenever a request arrives, if the token is left in the bucket, then the request will be executed and one token will be removed from the bucket each time
maintains a timed token generator which generates the token at a fixed rate and fills the bucket

```js
class TokenBucket {
  constructor(capacity, refillPerSec) {
    this.capacity = capacity;
    this.tokens = capacity;
    this.refillPerSec = refillPerSec;
    this.last = Date.now();
  }
  tryTake() {
    const now = Date.now();
    this.tokens = Math.min(this.capacity, this.tokens + ((now - this.last) / 1000) * this.refillPerSec);
    this.last = now;
    if (this.tokens >= 1) { this.tokens -= 1; return true; }
    return false; // caller should respond 429
  }
}
```

### Leak Bucket Algorithm :
 based on the methodology of how a bucket having hole at the bottom leaks the water, each request will be filled to the bucket if the bucket having space, else it will be rejected. and request will be processed sequentially by the backend. 

### 8.3 Idempotency keys

**Problem:** a payment POST times out — did it go through? Retrying might charge twice.

**Solution:** the client sends a unique `Idempotency-Key: <uuid>` header. The server stores the result for that key; a repeat request with the same key returns the stored result instead of creating a second payment. Same idea as webhook event-id de-duplication (section 3.2).

```js
await fetch("/v1/payments", {
  method: "POST",
  headers: { "Content-Type": "application/json", "Idempotency-Key": crypto.randomUUID() },
  body: JSON.stringify({ amount: 49900 })
});
```

### 8.4 Debounce and throttle

- **Debounce** — run only after the user _stops_ triggering for X ms (search box: call the API once after typing stops).
- **Throttle** — run at most once every X ms (scroll/resize handlers).

```js
function debounce(fn, ms) {
  let timer;                          // closure keeps the timer
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  };
}
searchInput.addEventListener("input", debounce(e => search(e.target.value), 300));
```

### 8.5 Pagination

APIs return large lists in pages. **Offset** (`?page=3&limit=50`) is simple but can skip/duplicate items when data changes; **cursor** (`?after=abc123`) is stable. Customer says "API returns only 100 records" → they didn't follow pagination.

```js
async function fetchAll(url) {
  const all = [];
  let next = url;
  while (next) {
    const res = await fetch(next);
    const page = await res.json();
    all.push(...page.data);
    next = page.next_url;   // null on last page
  }
  return all;
}
```

### 8.6 Binary search debugging (bisecting)

When "it worked last week": find the change that broke it by halving the search space — test the middle version, keep the half that contains the bug, repeat. `git bisect` automates this over commits. Same idea for config: disable half the plugins/extensions, see if the bug remains, repeat. Time: O(log n) checks instead of O(n).

### 8.7 Caching and timeouts (one-liners)

- **Caching** stores responses to avoid repeat calls; bugs = stale data → check `Cache-Control`, CDN, browser cache.
- **Timeouts** — always set one; use `AbortController` with fetch:

```js
const ctrl = new AbortController();
setTimeout(() => ctrl.abort(), 5000);
await fetch(url, { signal: ctrl.signal }); // throws AbortError after 5 s
```

## 9. Likely interview questions and revision checklist

Practise answering each in 3–5 sentences, out loud. Ordered by how often they come up.

### 9.1 Scenario questions (most common)

1. **A customer says our API returns 401. How do you troubleshoot?** → Check the key/token is sent, in the right header with the right prefix (`Bearer`), not expired, for the right environment (test vs live), not revoked. Decode the JWT to check `exp`. Reproduce in Postman.
2. **The API works in Postman but not in the customer's web app.** → Likely CORS (check console for "blocked by CORS policy"), or the app sends different headers/body. Compare the Network-tab request with the Postman one; Copy as cURL.
3. **Customer isn't receiving webhooks.** → Walk through the table in 3.5: delivery logs, URL, environment, reachability, response code, timeout, signature.
4. **Customer gets intermittent 429s / 503s.** → Rate limit or overload; implement backoff with jitter, respect `Retry-After`, reduce calls (cache, batch, webhooks).
5. **A button on the customer's site does nothing.** → Console errors, check element exists when script runs (`defer`), check if an overlay covers it (z-index), check listener is attached, check for extension interference (Incognito).
6. **Duplicate orders were created.** → Retries without idempotency, or webhook processed twice. Fix with idempotency keys / event-id de-dup.
7. **Page is slow.** → Network tab waterfall, large payloads, many requests, slow API (TTFB), Lighthouse.
8. **How would you explain a technical issue to a non-technical customer?** → Cause in one sentence, impact, what they need to do, when it will be fixed. No jargon.
9. **You don't know the answer — what do you do?** → Acknowledge, set a time, research docs/logs/colleagues, escalate with a clear report, follow up.

### 9.2 Theory questions

- What is an API? REST vs SOAP vs GraphQL? (REST: URLs + HTTP methods, JSON; SOAP: XML + strict contract; GraphQL: one endpoint, client chooses fields.)
- HTTP vs HTTPS? (HTTPS = HTTP encrypted with TLS.)
- What happens when you type a URL in the browser? (DNS lookup → TCP + TLS handshake → HTTP request → server response → browser parses HTML, loads CSS/JS → renders.)
- PUT vs PATCH; GET vs POST; 401 vs 403; 500 vs 502 vs 504.
- What is a webhook? Webhook vs polling vs WebSocket? (WebSocket = a two-way connection kept open.)
- How do you secure a webhook? (HTTPS, HMAC signature, timestamp, idempotency.)
- Cookies vs localStorage vs sessionStorage.
- What is CORS and why does it exist?
- `==` vs `===`; `let` vs `var` vs `const`; what is a closure; what is a promise; event loop output question.
- `async` vs `defer`; semantic HTML; `display: none` vs `visibility: hidden`; flexbox vs grid; specificity.
- What is a browser extension made of? Content script vs service worker? What changed in Manifest V3?

### 9.3 Small coding tasks you may get

- Write a `fetch` call with error handling (section 1.9).
- Parse a JSON response and print fields / filter an array (section 2.5).
- Write `debounce` (8.4) or a retry function (8.1).
- Fix a buggy snippet: missing `await`, missing `JSON.stringify`, `this` lost in a callback, script running before DOM.
- Center a div: `display:flex; justify-content:center; align-items:center;`.
- Verify an HMAC signature (3.3).

### 9.4 Last-day revision checklist

- Recite the status-code table, especially 400/401/403/404/409/422/429/500/502/503/504
- Explain the 7-step API troubleshooting method from memory
- Explain CORS and preflight in 30 seconds
- Write `fetch` + `try/catch` + `res.ok` without looking
- Predict event-loop output (A D C B)
- Explain webhooks: register → event → POST → verify HMAC → 200 → process; retries + idempotency
- Name extension parts: manifest, service worker, content script, popup; messaging
- Know DevTools: Network filters, Preserve log, Copy as cURL, HAR
- Prepare one story from your own projects where you debugged an API or integration issue (STAR: Situation, Task, Action, Result)
- Prepare 2 questions for the interviewer (tools they use, ticket volume, escalation process)