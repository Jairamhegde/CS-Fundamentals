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

### 4.2 Things to remember

- **Element vs tag vs attribute:**  Element is a whole structure which consist of `tag`,`attributes and data`, where as  tag is a markup which defines the starting and ending of a element
  attribute is the information specified inside the tag
	
- **Block vs inline:** block (`div, p, h1, section`) starts on a new line, full width;
	inline  (`span, a, strong`) sits within text. does not takes extra space.
	
- **Semantic HTML:** `header, nav, main, article, section, footer, button` : convey meaning of the elements , lead to  better accessibility and SEO than `div` everywhere.

- **`id` vs `class`:** `id` is unique per page; `class` can repeat. Duplicate ids break `getElementById` and labels.

- **`data-*`  : attributes:**  Stores extra information inside elements , that JavaScript can use later for operations. 
- `ARIA` : used when semantic tags are not enough to describe the behavior. we use aria as custom semantics

- **Accessibility:** `alt` text on images, `<label for>` on inputs, real `<button>` instead of clickable `div`, ARIA only when needed.

### 4.3 Script loading 

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