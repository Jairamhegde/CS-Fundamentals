
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

## 2. JavaScript core

**What & why:** JavaScript (JS) is the programming language of the web. It runs in every browser (making pages interactive) and on servers via Node.js. In support you will read customer code, explain console errors, and write small scripts to reproduce bugs — so focus on **async code, errors, and the DOM**.

### 2.1 Variables: var, let, const

|   |   |   |   |
|---|---|---|---|
|Keyword|Scope|Re-assign?|Hoisting|
|`var`|Function|Yes|Hoisted, starts as `undefined`|
|`let`|Block `{ }`|Yes|Hoisted but unusable before the line ("temporal dead zone")|
|`const`|Block `{ }`|No (but objects inside can still change)|Same as let|

```js
const user = { name: "JK" };
user.name = "Jairam";   // allowed — the object changed, not the binding
// user = {};            // TypeError: Assignment to constant variable
```

### 2.2 Data types and comparisons

- **Primitives:** string, number, boolean, `null`, `undefined`, bigint, symbol.
- **Objects:** objects, arrays, functions, dates.
- `null` = "intentionally empty"; `undefined` = "not set yet".
- `typeof null` returns `"object"` (a famous old bug).
- **Always use `===`** (strict, no type conversion). `==` converts types: `0 == ""` is `true`, `0 === ""` is `false`.
- **Falsy values:** `false, 0, "", null, undefined, NaN`. Everything else is truthy (including `[]` and `{}`).

### 2.3 Functions, arrow functions and `this`

```js
function add(a, b) { return a + b; }      // declaration (hoisted)
const add2 = (a, b) => a + b;             // arrow function

const obj = {
  name: "widget",
  normal() { return this.name; },          // this = obj
  arrow: () => this?.name                  // arrow has NO own this → undefined here
};
```

**`this` rule:** in a normal function, `this` is the object _before the dot_ when called. Arrow functions take `this` from the surrounding code. Losing `this` (e.g. passing `obj.normal` as a callback) is a common bug — fix with `.bind(obj)` or an arrow function.

### 2.4 Scope and closures

**Closure** = a function that remembers the variables around it, even after the outer function has finished.

```js
function makeCounter() {
  let count = 0;            // private variable
  return () => ++count;     // inner function "closes over" count
}
const next = makeCounter();
next(); // 1
next(); // 2
```

Used for private data, event handlers, and debounce/throttle (section 8).

### 2.5 Arrays and objects you must know

```js
const orders = [{ id: 1, total: 500 }, { id: 2, total: 1500 }];

orders.map(o => o.id);                 // [1, 2]       transform each
orders.filter(o => o.total > 1000);    // [{id:2...}]  keep some
orders.find(o => o.id === 2);          // {id:2...}    first match
orders.reduce((sum, o) => sum + o.total, 0); // 2000   combine

const { id, total } = orders[0];       // destructuring
const copy = { ...orders[0], total: 600 }; // spread: copy + override
const safe = response?.data?.items ?? [];  // optional chaining + default
```

### 2.6 Asynchronous JavaScript (most important)

**Why:** JS runs on one thread. Slow work (network, timers, files) is done _in the background_; JS continues and handles the result later.

**Three styles, same idea:**

```js
// 1. Callback (old)
setTimeout(() => console.log("later"), 1000);

// 2. Promise — an object that will hold a future value: pending → fulfilled / rejected
fetch("/api/user")
  .then(res => res.json())
  .then(user => console.log(user))
  .catch(err => console.error(err));

// 3. async/await — cleaner syntax over promises
async function loadUser() {
  try {
    const res = await fetch("/api/user");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.error("Failed:", err.message);
  }
}
```

**Running in parallel:**

```js
const [user, orders] = await Promise.all([getUser(), getOrders()]); // fails if ANY fails
const results = await Promise.allSettled([a(), b()]);               // never fails; check each status
```

**Common bugs:** forgetting `await` (you get a `Promise {<pending>}` instead of data), no `.catch` ("Uncaught (in promise)" error), using `await` inside `forEach` (it doesn't wait — use `for...of`).

### 2.7 The event loop (theory question)

Order of execution: **1) all synchronous code → 2) microtasks (promise callbacks) → 3) macrotasks (setTimeout, events)**.

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
// Output: A D C B
```

**Explanation:** A and D are synchronous. The promise callback (C) is a microtask and runs before the timer (B), even with a 0 ms delay.

### 2.8 The DOM and events

**DOM (Document Object Model)** = the browser's tree of objects built from the HTML. JS reads and changes the page through it.

```js
const btn = document.querySelector("#buy");        // select (null if not found!)
btn.textContent = "Buy now";                         // change text
btn.classList.add("active");                         // change style via class
btn.addEventListener("click", (e) => {
  e.preventDefault();                                // stop default (e.g. form submit)
  console.log("clicked", e.target);
});

// Event delegation: one listener on the parent handles many children (even ones added later)
document.querySelector("#list").addEventListener("click", (e) => {
  if (e.target.matches("li")) console.log("item", e.target.dataset.id);
});

// Run code only after the HTML is parsed
document.addEventListener("DOMContentLoaded", init);
```

**Event bubbling:** an event fires on the clicked element, then on each parent up to `document`. `e.stopPropagation()` stops it.

### 2.9 Errors you will see in tickets

|   |   |   |
|---|---|---|
|Error|Meaning|Usual fix|
|`TypeError: Cannot read properties of undefined (reading 'x')`|You did `obj.x` where `obj` is undefined|Check the data exists; use `obj?.x`|
|`TypeError: x is not a function`|Calling something that isn't a function|Typo, wrong import, script not loaded|
|`ReferenceError: x is not defined`|Variable never declared / script order|Load scripts in the right order|
|`SyntaxError: Unexpected token '<'` in JSON|Got HTML (an error page) when JSON was expected|Check URL and status code|
|`Uncaught (in promise)`|A rejected promise had no catch|Add try/catch or `.catch`|
|`Failed to fetch`|Network failure, CORS block, or bad URL|Check Network tab and console|

### 2.10 Storage, JSON and modules (quick facts)

- `JSON.stringify(obj)` → string; `JSON.parse(str)` → object (throws on invalid JSON).
- **localStorage** — 5–10 MB, survives restarts, same origin only. **sessionStorage** — cleared when the tab closes. **Cookies** — small, sent to the server with each request.
- **Modules:** `import { x } from "./a.js"` / `export`. In Node, older code uses `require()`.
- Debugging: `console.log`, `console.table`, `debugger;` statement, breakpoints in the Sources tab.

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
- **Idempotency** — providers deliver _at least once_, so you may get the same event twice. Store processed `event.id`s and skip duplicates.
- **Respond fast** — return 200 within a few seconds; do heavy work later. Slow handlers cause timeouts → retries → duplicates.
- **Ordering** — events can arrive out of order; use timestamps or re-fetch the latest state from the API.

### 3.3 Code: a safe webhook receiver (Node.js + Express)

```js
const express = require("express");
const crypto = require("crypto");
const app = express();

const SECRET = process.env.WEBHOOK_SECRET;
const processed = new Set(); // in production: a database table

// express.raw keeps the body as raw bytes — needed for the signature check
app.post("/webhooks/payments", express.raw({ type: "application/json" }), (req, res) => {
  const received = req.get("X-Signature") || "";
  const expected = crypto.createHmac("sha256", SECRET).update(req.body).digest("hex");

  // timingSafeEqual avoids timing attacks; lengths must match first
  const valid = received.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(received), Buffer.from(expected));
  if (!valid) return res.status(401).send("Invalid signature");

  const event = JSON.parse(req.body.toString());

  if (processed.has(event.id)) return res.sendStatus(200); // duplicate → ignore, but still 200
  processed.add(event.id);

  res.sendStatus(200);          // acknowledge quickly
  handleEventLater(event);      // then do the slow work (DB update, email...)
});

function handleEventLater(event) {
  if (event.type === "payment.succeeded") {
    console.log("Mark order paid:", event.data.payment_id);
  }
}

app.listen(3000);
```

**Explanation:** raw body → verify HMAC → reject if invalid (401) → skip duplicates → reply 200 → process. Say these five steps in the interview and you will sound experienced.

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

## 4. HTML essentials

**What & why:** HTML (HyperText Markup Language) describes the **structure and meaning** of a page — headings, paragraphs, forms, buttons, images. The browser reads it and builds the DOM. In support, you'll inspect a customer's page to see why an embed, form or script isn't working.

### 4.1 Page skeleton

```html
<!DOCTYPE html>                <!-- standards mode; missing it = odd layout bugs -->
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1"> <!-- mobile layout -->
  <title>Store</title>
  <link rel="stylesheet" href="styles.css">
  <script src="app.js" defer></script>
</head>
<body>
  <header><nav>...</nav></header>
  <main><h1>Products</h1></main>
  <footer>...</footer>
</body>
</html>
```

### 4.2 Things interviewers ask

- **Element vs tag vs attribute:** `<a href="/x">Link</a>` — `a` is the tag, `href` an attribute, the whole thing an element.
- **Block vs inline:** block (`div, p, h1, section`) starts on a new line, full width; inline (`span, a, strong`) sits within text.
- **Semantic HTML:** `header, nav, main, article, section, footer, button` describe meaning → better accessibility and SEO than `div` everywhere.
- **`id` vs `class`:** `id` is unique per page; `class` can repeat. Duplicate ids break `getElementById` and labels.
- **`data-*` attributes:** store custom data: `<li data-id="42">` → JS reads `el.dataset.id`.
- **Accessibility:** `alt` text on images, `<label for>` on inputs, real `<button>` instead of clickable `div`, ARIA only when needed.

### 4.3 Script loading (a frequent cause of bugs)

|   |   |   |   |
|---|---|---|---|
|How|Downloads|Runs|Use for|
|`<script src>` in head|Blocks parsing|Immediately|Rarely — slows page, DOM not ready|
|`<script defer>`|In parallel|After HTML parsed, in order|App scripts that use the DOM|
|`<script async>`|In parallel|As soon as downloaded, any order|Independent scripts (analytics)|
|`<script type="module">`|In parallel|Deferred by default|Modern ES modules|

**Typical ticket:** "`Cannot read properties of null (reading 'addEventListener')`" → the script ran before the element existed. Fix: `defer`, or move the script to the end of `<body>`.

### 4.4 Forms

```html
<form action="/api/signup" method="POST">
  <label for="email">Email</label>
  <input id="email" name="email" type="email" required>
  <button type="submit">Sign up</button>
</form>
```

- Inputs are sent using their `name` (not `id`) — a missing `name` means the field never reaches the server.
- A `<button>` inside a form defaults to `type="submit"` and reloads the page; use `type="button"` or `e.preventDefault()` in JS.
- Default form encoding is `application/x-www-form-urlencoded`; file uploads need `enctype="multipart/form-data"`.

### 4.5 Embeds and iframes

`<iframe src="https://widget.com/chat">` loads another site inside yours. It may refuse to load if that site sends `X-Frame-Options: DENY` or a `Content-Security-Policy: frame-ancestors` rule. **CSP (Content Security Policy)** headers can also block third-party scripts — check the console for "Refused to load..." messages.

## 5. CSS essentials

**What & why:** CSS (Cascading Style Sheets) controls how HTML **looks and is laid out** — colours, fonts, spacing, positions, responsive design. Support tickets: "our widget is hidden", "button overlaps", "looks broken on mobile", "your styles changed our site".

### 5.1 Selectors and specificity

```css
p { color: black; }              /* element   — weakest */
.price { color: green; }         /* class     */
#total { color: red; }           /* id        */
/* inline style="" beats all of these; !important beats inline */
```

**Specificity order:** inline style > id > class / attribute / pseudo-class > element. If equal, the **later rule wins** (that is the "cascade"). When a customer says "my CSS doesn't apply", check in DevTools → Elements → Styles: the rule will be shown **crossed out** with the winning rule above it.

### 5.2 Box model

Every element is a box: **content → padding → border → margin**.

```css
* { box-sizing: border-box; } /* width now INCLUDES padding + border — avoids overflow surprises */
```

### 5.3 Display and layout

- `display: none` removes the element (no space); `visibility: hidden` hides it but keeps its space; `opacity: 0` invisible but still clickable.
- **Flexbox** — one-dimensional layout (row _or_ column):

```css
.toolbar { display: flex; justify-content: space-between; align-items: center; gap: 8px; }
```

- **Grid** — two-dimensional (rows _and_ columns):

```css
.cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px; }
```

### 5.4 Positioning and z-index

|   |   |
|---|---|
|`position`|Behaviour|
|`static`|Default, normal flow|
|`relative`|Moved relative to itself; becomes the reference for absolute children|
|`absolute`|Placed relative to nearest positioned ancestor; removed from flow|
|`fixed`|Placed relative to the screen; stays when scrolling (chat bubbles)|
|`sticky`|Normal until a scroll point, then sticks|

**z-index** controls which element is on top, but only works on positioned elements and only _within the same stacking context_. A parent with `transform`, `opacity < 1` or its own z-index creates a new stacking context — a classic reason a widget "won't come on top" even with `z-index: 99999`.

### 5.5 Responsive design

```css
.sidebar { width: 300px; }
@media (max-width: 768px) {   /* screens 768px wide or less */
  .sidebar { display: none; }
}
```

Units: `px` fixed; `%` of parent; `rem` relative to root font size; `vh/vw` % of the screen. Remember the viewport `<meta>` tag from section 4.1.

### 5.6 Debugging CSS quickly

1. Right-click → **Inspect** the element.
2. Check **Styles** (which rules apply, which are crossed out) and **Computed** (final values).
3. Look at the box-model diagram for unexpected margin/padding.
4. Toggle rules on/off; add `outline: 1px solid red` to see boxes.
5. Check for style leaks: third-party widgets should scope their classes (prefixes or Shadow DOM) so the host page's CSS doesn't break them.

## 6. Browser extensions

**What & why:** A browser extension is a small program (HTML + CSS + JS) that adds features to Chrome, Edge or Firefox — e.g. Grammarly, password managers, ad blockers, a company's own Chrome extension. In this role you may support a product that _is_ an extension, or debug customer issues _caused by_ extensions (an ad blocker blocking your script, an extension changing the page).

> Note: "Extensions" in a JD can also mean platform add-ons/plugins (e.g. extending a SaaS product via its API). The browser-extension meaning is the most common; the API skills from sections 1 and 3 cover the other meaning.

### 6.1 The parts of an extension (Manifest V3)

|   |   |   |   |
|---|---|---|---|
|Part|What it is|Can access the web page?|Can use most `chrome.*` APIs?|
|`manifest.json`|Config file: name, version, permissions, files|—|—|
|Service worker (background)|Event-driven script; no UI; starts and stops automatically|No|Yes|
|Content script|JS injected into web pages|Yes (reads/edits the DOM)|Only a few (messaging, storage)|
|Popup|Small HTML page when you click the toolbar icon|No|Yes|
|Options page|Settings page|No|Yes|

**Manifest V3 (MV3)** is Chrome's current format (MV2 is phased out). Key changes: background pages → **service workers** (no persistent memory, no DOM), no remotely hosted code, `webRequest` blocking → `declarativeNetRequest`.

### 6.2 Minimal example

```json
{
  "manifest_version": 3,
  "name": "Price Highlighter",
  "version": "1.0",
  "permissions": ["storage"],
  "host_permissions": ["https://*.shop.com/*"],
  "background": { "service_worker": "background.js" },
  "content_scripts": [
    { "matches": ["https://*.shop.com/*"], "js": ["content.js"] }
  ],
  "action": { "default_popup": "popup.html" }
}
```

```js
// content.js — runs inside shop.com pages
document.querySelectorAll(".price").forEach(el => el.style.background = "yellow");
chrome.runtime.sendMessage({ type: "PRICES_FOUND", count: document.querySelectorAll(".price").length });
```

```js
// background.js — the service worker
chrome.runtime.onMessage.addListener((msg, sender, sendResponse) => {
  if (msg.type === "PRICES_FOUND") {
    chrome.storage.local.set({ lastCount: msg.count });
    sendResponse({ ok: true });
  }
  return true; // keep the channel open if you respond asynchronously
});
```

**Explanation:** the content script can see the page but has limited powers, so it **messages** the service worker, which has full extension APIs (storage, tabs, network). This message passing is the core extension concept.

### 6.3 Important concepts

- **Permissions** — `permissions` (APIs like `storage`, `tabs`) and `host_permissions` (which sites). Missing permission = silent failure or an error in the console.
- **Isolated world** — content scripts share the page's DOM but **not its JavaScript variables**. They can't directly read `window.myAppConfig` of the page.
- **Storage** — use `chrome.storage` (not `localStorage`) in service workers. Service workers can be shut down at any time, so global variables are lost.
- **CORS in extensions** — requests from the service worker to hosts in `host_permissions` are allowed; requests from content scripts follow the page's CORS rules.
- **CSP** — MV3 forbids `eval` and inline scripts in extension pages.

### 6.4 Debugging an extension

1. Go to `chrome://extensions` → turn on **Developer mode** → **Load unpacked** to test a local folder.
2. **Errors** button on the extension card shows runtime errors.
3. Service worker: click **"Inspect views: service worker"** to open its own DevTools console.
4. Popup: right-click the popup → Inspect.
5. Content script: the normal page DevTools → Console → pick the extension in the context dropdown.
6. After changing code, click **Reload** on the extension card and refresh the page.

### 6.5 Common tickets

|   |   |
|---|---|
|Symptom|Likely cause|
|Feature works, then stops after a while|MV3 service worker was terminated; state kept in a global variable|
|Content script does nothing on a site|URL not in `matches`, page loaded before install (refresh), site is in an iframe|
|"Could not establish connection. Receiving end does not exist."|Message sent to a tab with no content script loaded|
|Our website breaks for some users only|An installed extension (ad blocker, translator) is blocking scripts or changing the DOM — ask user to test in Incognito with extensions off|
|Extension can't call our API|Missing `host_permissions`, or call made from content script and blocked by CORS|

## 7. Browser DevTools and support skills

**What & why:** DevTools (F12 or Ctrl+Shift+I) is the support engineer's main instrument. Interviewers often give a scenario and want to hear _which tab you open and what you look for_. The second half of this section covers the soft skills that make up half the job.

### 7.1 DevTools tabs

|   |   |   |
|---|---|---|
|Tab|Use it to|Support tip|
|Elements|Inspect HTML and CSS live|Find why an element is hidden / mis-styled|
|Console|See JS errors, run JS|Red errors first; read the file and line number|
|Network|Every request: status, headers, payload, response, timing|Filter **Fetch/XHR**; tick **Preserve log** to keep requests across page loads|
|Sources|View code, set breakpoints|Step through to see variable values|
|Application|localStorage, cookies, service workers, cache|Clear site data to rule out stale cache|
|Performance / Lighthouse|Speed and best-practice audits|Slow-page tickets|

**Network tab tricks:** right-click a request → **Copy as cURL** (replay it outside the browser); **Save all as HAR** — a HAR file is a recording of all network traffic that customers can send you. _Warning: HAR files contain cookies and tokens — ask customers to remove them or treat as sensitive._ Use throttling ("Slow 3G") to reproduce slow-network bugs; "Disable cache" to rule out caching.

### 7.2 Quick isolation checklist (for any "it doesn't work" ticket)

1. Hard refresh (Ctrl+Shift+R) / clear cache.
2. Incognito window (no extensions, no cookies).
3. Another browser / device / network.
4. Another user account (permissions?).
5. Status page of the service — is there an outage?
6. What changed recently? (deploy, config, key rotation, browser update)

### 7.3 Handling a ticket

1. **Acknowledge** quickly and set expectations.
2. **Gather info:** steps to reproduce, expected vs actual, screenshots, console errors, HAR, request ids, account/environment, time with timezone.
3. **Reproduce** in your own environment.
4. **Prioritise by impact:** P1 = production down / many users; P4 = question or cosmetic.
5. **Resolve or escalate** — escalate to engineering with a clear bug report: summary, steps, expected/actual, evidence, impact, what you already ruled out.
6. **Communicate** in plain language; avoid blame; give a workaround if possible.
7. **Close the loop** and add to the knowledge base.

**Useful terms:** SLA (Service Level Agreement — promised response/resolution times), L1/L2/L3 support tiers, RCA (root cause analysis), KB (knowledge base), escalation, workaround.

### 7.4 Example customer reply

> Hi Priya, thanks for the details and the HAR file. The requests to `/v1/orders` are returning **401** because the API key used belongs to the _test_ environment, while the URL is the _live_ one. Please use your live key from Dashboard → Developers → API keys. I've confirmed this works with a test call on our side. Let me know if you still see the error and I'll stay on it.

Good replies: name the cause, the evidence, the exact fix, and the next step.

### 7.5 Basic Git and Linux you may be asked

- Git: `clone, status, add, commit, push, pull, branch, checkout, log`.
- Linux: `ls, cd, cat, grep "error" app.log, tail -f app.log, ps, curl, ping, nslookup/dig` (DNS).

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

**Token bucket algorithm:** a bucket holds up to N tokens and refills at a fixed rate. Each request takes one token; empty bucket = reject (429). It allows short bursts but caps the average rate. (Other algorithms: fixed window, sliding window, leaky bucket.)

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

**Client side fixes for 429:** backoff, queue requests, batch calls, cache results, use webhooks instead of polling.

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