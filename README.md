# 📖 CWE Chronicles
A structured breakdown of the **2025 CWE Top 25 Most Dangerous Software Weaknesses** — written by a final-year CSE student who decided to actually read the list properly.

---

## Why This Exists

The CWE Top 25 is one of the most referenced lists in existence. Attackers know it. Security teams live by it. Yet most developers have only skimmed the titles.

The result is the **CWE Top 25** — a list that tells you exactly where software fails, and exactly where attackers look first.

This repo is my attempt to work through every entry on that list — not surface-level summaries, but actual breakdowns: what the vulnerability is, why it keeps happening, what real attacks look like, and how to write code that doesn't fall into the same trap.

I'm a final-year CSE student. I'm not a security researcher. I'm someone who decided to read the list properly and write about it publicly — because the best way to understand something is to explain it to someone else.

---

## 2025 CWE Top 25 — Progress

| # | CWE | Name | Status | Write-up |
|---|-----|------|--------|----------|
| 1 | CWE-79 | Cross-Site Scripting (XSS) | ✅ Published | [Read](./CWE-79-Cross-Site-Scripting.md) |
| 2 | CWE-89 | SQL Injection | ✅ Published | [Read](./CWE-89-SQL-Injection.md) |
| 3 | CWE-352 | Cross-Site Request Forgery (CSRF) | ✅ Published | [Read](./CWE-352-Cross-Site-Request-Forgery.md) |
| 4 | CWE-22 | Path Traversal | ⏳ Planned | — |
| 5 | CWE-125 | Out-of-bounds Read | ⏳ Planned | — |
| 6 | CWE-787 | Out-of-bounds Write | ⏳ Planned | — |
| 7 | CWE-20 | Improper Input Validation | ⏳ Planned | — |
| 8 | CWE-416 | Use After Free | ⏳ Planned | — |
| 9 | CWE-862 | Missing Authorization | ⏳ Planned | — |
| 10 | CWE-78 | OS Command Injection | ⏳ Planned | — |
| 11 | CWE-190 | Integer Overflow | ⏳ Planned | — |
| 12 | CWE-306 | Missing Authentication | ⏳ Planned | — |
| 13 | CWE-502 | Deserialization of Untrusted Data | ⏳ Planned | — |
| 14 | CWE-77 | Command Injection | ⏳ Planned | — |
| 15 | CWE-119 | Buffer Overflow | ⏳ Planned | — |
| 16 | CWE-798 | Use of Hard-coded Credentials | ⏳ Planned | — |
| 17 | CWE-918 | Server-Side Request Forgery (SSRF) | ⏳ Planned | — |
| 18 | CWE-476 | NULL Pointer Dereference | ⏳ Planned | — |
| 19 | CWE-434 | Unrestricted File Upload | ⏳ Planned | — |
| 20 | CWE-94 | Code Injection | ⏳ Planned | — |
| 21 | CWE-400 | Uncontrolled Resource Consumption | ⏳ Planned | — |
| 22 | CWE-295 | Improper Certificate Validation | ⏳ Planned | — |
| 23 | CWE-732 | Incorrect Permission Assignment | ⏳ Planned | — |
| 24 | CWE-601 | Open Redirect | ⏳ Planned | — |
| 25 | CWE-611 | XML External Entity (XXE) Injection | ⏳ Planned | — |

---

## 📁 File Structure

```
cwe-chronicles/
├── README.md
├── CWE-79-Cross-Site-Scripting.md
├── CWE-89-SQL-Injection.md
├── CWE-352-Cross-Site-Request-Forgery.md
├── (more entries as published)
└── ...
```

---

## 📝 What Each Write-up Covers

Every article in this series follows the same structure:

1. **The Vulnerability** — What it is, in plain English
2. **The Mechanics** — How it works, with code examples
3. **Real-World Impact** — Actual CVE examples and attack scenarios
4. **Why It Still Happens** — Common mistakes developers make
5. **Defense Strategies** — Practical mitigation techniques
6. **Defensive Code** — Working examples of secure implementations
7. **Testing & Detection** — How to find and test for it

---

## 🔑 Key Features

- **No fluff.** No "vulnerability 101" preamble. Straight to the point.
- **Code-heavy.** Vulnerable code → secure code. You'll see the difference.
- **Real examples.** Based on actual CVEs and published exploits, not hypotheticals.
- **Developer-focused.** Written for people who build things, not just people who read security papers.
- **Updated regularly.** As each entry is completed, the README updates automatically.

---

## 🎯 Who This Is For

- **Final-year CSE/IT students** preparing for careers in security or backend development
- **Junior developers** who want to understand common vulnerabilities without drowning in academic jargon
- **Hiring teams** looking for evidence of security fundamentals understanding
- **Security enthusiasts** who want a structured reference on the Top 25

---

## 🔗 References

- [MITRE CWE Top 25 (2025)](https://cwe.mitre.org/top25/archive/2025/2025_cwe_top25.html)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CISA Known Exploited Vulnerabilities](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [Common Weakness Enumeration (CWE)](https://cwe.mitre.org/)

---

## 📊 Stats

- **Total Entries:** 25
- **Published:** 3
- **In Progress:** 22
- **Expected Timeline:** One entry every 2-3 weeks

---

## 💡 Contributing

Found an error? Have a suggestion? Security researchers, practitioners, and fellow students — feedback is welcome. Open an issue or reach out.

---

## 📄 License

All content is original and available for educational use. Attribution appreciated.

---

*Last Updated: January 2026*  
*Next write-up: CWE-22 (Path Traversal)*
