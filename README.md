# 📖 CWE Chronicles

A structured breakdown of the **2025 CWE Top 25 Most Dangerous Software Weaknesses** — written by a final-year CSE student who decided to actually read the list properly.

---

## Why This Exists

The CWE Top 25 is one of the most referenced lists in existence. The result is the **CWE Top 25** — a list that tells you exactly where software fails, and exactly where attackers look first.

This repo is my attempt to work through every entry on that list — not surface-level summaries, but actual breakdowns: what the vulnerability is, why it keeps happening, what real attacks look like, and how to write code that doesn't fall into the same trap.

I'm a final-year CSE student. I'm not a security researcher. I'm someone who decided to read the list properly and write about it publicly — because the best way to understand something is to explain it to someone else.

---

## 2025 CWE Top 25 — Progress

| # | CWE | Name | Status |
|---|-----|------|--------|
| 1 | CWE-79 | Cross-Site Scripting (XSS) | ✅ Published |
| 2 | CWE-89 | SQL Injection | ✅ Published |
| 3 | CWE-352 | Cross-Site Request Forgery (CSRF) | 🔜 Coming next |
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
| 16 | CWE-798 | Use of Hard-coded Credentials | ⏳ Planned |
| 17 | CWE-918 | Server-Side Request Forgery (SSRF) | ⏳ Planned |
| 18 | CWE-476 | NULL Pointer Dereference | ⏳ Planned |
| 19 | CWE-434 | Unrestricted File Upload | ⏳ Planned |
| 20 | CWE-94 | Code Injection | ⏳ Planned |
| 21 | CWE-400 | Uncontrolled Resource Consumption | ⏳ Planned |
| 22 | CWE-295 | Improper Certificate Validation | ⏳ Planned |
| 23 | CWE-732 | Incorrect Permission Assignment | ⏳ Planned |
| 24 | CWE-601 | Open Redirect | ⏳ Planned |
| 25 | CWE-611 | XML External Entity (XXE) Injection | ⏳ Planned |

---

## 📁 File Structure

```
cwe-chronicles/
├── README.md
├── CWE-79-Cross-Site-Scripting.md
├── CWE-89-SQL-Injection.md
└── ...
```

---

## 🔗 References

- [MITRE CWE Top 25 (2025)](https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CISA Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)

---

*Updated as each write-up is published. All entries based on official CWE definitions and real CVE examples.*
