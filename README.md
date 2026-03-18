# 📓 CWE Chronicles

> A CSE fresher's deep-dive into the 2025 MITRE CWE Top 25 — written to actually understand it, not just pass an exam.

---

## What is this?

Every year, MITRE analyzes thousands of real-world CVEs and ranks the most dangerous software weaknesses in existence. The result is the **CWE Top 25** — a list that tells you exactly where software fails, and exactly where attackers look first.

This repo is my attempt to work through every entry on that list — not surface-level summaries, but actual breakdowns: what the vulnerability is, why it keeps happening, what real attacks look like, and how to write code that doesn't fall into the same trap.

I'm a final-year CSE student. I'm not a security researcher. I'm someone who decided to read the list properly and write about it publicly — because the best way to understand something is to explain it to someone else.

---

## 2025 CWE Top 25 — Progress

| # | CWE | Name | Status |
|---|-----|------|--------|
| 1 | CWE-79 | Cross-Site Scripting (XSS) | ✅ Published |
| 2 | CWE-89 | SQL Injection | 🔜 Coming next |
| 3 | CWE-352 | Cross-Site Request Forgery (CSRF) | ⏳ Planned |
| 4 | CWE-22 | Path Traversal | ⏳ Planned |
| 5 | CWE-125 | Out-of-bounds Read | ⏳ Planned |
| 6 | CWE-787 | Out-of-bounds Write | ⏳ Planned |
| 7 | CWE-20 | Improper Input Validation | ⏳ Planned |
| 8 | CWE-416 | Use After Free | ⏳ Planned |
| 9 | CWE-862 | Missing Authorization | ⏳ Planned |
| 10 | CWE-78 | OS Command Injection | ⏳ Planned |
| 11 | CWE-190 | Integer Overflow | ⏳ Planned |
| 12 | CWE-306 | Missing Authentication | ⏳ Planned |
| 13 | CWE-502 | Deserialization of Untrusted Data | ⏳ Planned |
| 14 | CWE-77 | Command Injection | ⏳ Planned |
| 15 | CWE-119 | Buffer Overflow | ⏳ Planned |
| 16 | CWE-798 | Hard-coded Credentials | ⏳ Planned |
| 17 | CWE-918 | SSRF | ⏳ Planned |
| 18 | CWE-476 | NULL Pointer Dereference | ⏳ Planned |
| 19 | CWE-400 | Uncontrolled Resource Consumption | ⏳ Planned |
| 20 | CWE-611 | XXE Injection | ⏳ Planned |
| 21 | CWE-94 | Code Injection | ⏳ Planned |
| 22 | CWE-269 | Improper Privilege Management | ⏳ Planned |
| 23 | CWE-276 | Incorrect Default Permissions | ⏳ Planned |
| 24 | CWE-287 | Improper Authentication | ⏳ Planned |
| 25 | CWE-434 | Unrestricted File Upload | ⏳ Planned |

---

## How each post is structured

Every entry follows the same format so it's easy to read and reference:

- **What it is** — plain English explanation of the vulnerability
- **Why it keeps happening** — the structural / human reasons it persists
- **Real-world impact** — actual CVEs and known exploits from the wild
- **Code examples** — vulnerable vs. secure code side by side
- **How to fix it** — developer-focused, practical prevention steps
- **Key takeaway** — one thing to remember

---

## Who this is for

- **Fellow CS students** who want to understand security beyond textbook definitions
- **Junior developers** who want to write safer code from day one
- **Anyone prepping for security interviews** or certifications like CEH, OSCP, or CompTIA Security+

---

## Posts also on LinkedIn

Each entry in this repo is cross-posted as a LinkedIn article. If you prefer reading there, follow me at [linkedin.com/in/yourprofile](https://linkedin.com/in/tanmaymish).

---

## References & Sources

- [MITRE CWE Top 25 (2025)](https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html)
- [CISA Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [NVD — National Vulnerability Database](https://nvd.nist.gov/)

---

## Let's connect

If you're reading through the CWE Top 25 too, or just want to talk security and software — reach out on LinkedIn or drop an issue on this repo.

*New entry every week. Watch the repo to get notified.*

---

![GitHub last commit](https://img.shields.io/github/last-commit/yourusername/cwe-chronicles)
![GitHub stars](https://img.shields.io/github/stars/yourusername/cwe-chronicles?style=social)
