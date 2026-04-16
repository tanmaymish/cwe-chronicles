# 🛡️ SQL Injection (CWE-89) — Security Write-Up

> **CWE-89 | Improper Neutralization of Special Elements used in an SQL Command**  
> Likelihood of Exploit: **HIGH** | OWASP Top 10: **A03:2021 – Injection**

---

## 📌 What is SQL Injection?

SQL Injection (SQLi) is a critical web security vulnerability that occurs when **user-controlled input is embedded directly into SQL queries** without proper sanitization. This allows attackers to manipulate the query's logic — potentially reading, modifying, or deleting database data, bypassing authentication, or even executing system commands.

---

## ⚙️ How It Works

### Vulnerable Code Example (C#)
```csharp
string userName = ctx.getAuthenticatedUserName();
string query = "SELECT * FROM items WHERE owner = '" + userName + "'"
             + " AND itemname = '" + ItemName.Text + "'";
```

### Intended Query
```sql
SELECT * FROM items WHERE owner = 'wiley' AND itemname = 'laptop';
```

### Attacker Input
```
name' OR 'a'='a
```

### Resulting Malicious Query
```sql
SELECT * FROM items WHERE owner = 'wiley' AND itemname = 'name' OR 'a'='a';
-- This returns ALL rows in the table!
```

---

## 💥 Attack Scenarios

### 1. Authentication Bypass
```sql
-- Login form injection
' OR '1'='1' --
-- Logs in as the first user without knowing the password
```

### 2. Data Deletion
```sql
-- Input: name'; DELETE FROM items; --
SELECT * FROM items WHERE owner = 'wiley' AND itemname = 'name';
DELETE FROM items;
--'
```

### 3. OS Command Execution (MS SQL Server)
```sql
-- Input: '; exec master..xp_cmdshell 'dir' --
SELECT ITEM, PRICE FROM PRODUCT WHERE ITEM_CATEGORY='';
exec master..xp_cmdshell 'dir'
--' ORDER BY PRICE
```

### 4. Cookie / Header Manipulation (PHP)
```php
// Vulnerable code
$id = $_COOKIE["mid"];
mysql_query("SELECT MessageID, Subject FROM messages WHERE MessageID = '$id'");

// Attacker sets cookie to: 1432' or '1' = '1
// Result: returns ALL messages
```

---

## 🔥 Common Consequences

| Impact Area       | Description |
|-------------------|-------------|
| **Confidentiality** | Read sensitive data from the database |
| **Integrity**       | Modify or delete data |
| **Availability**    | Drop tables or corrupt data |
| **Authentication**  | Bypass login mechanisms |
| **Authorization**   | Escalate privileges or access others' data |
| **System Access**   | Execute OS commands via database functions |

---

## ✅ Mitigations

### ✔️ 1. Use Parameterized Queries / Prepared Statements *(Most Effective)*
```python
# Python (safe)
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

```java
// Java (safe)
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM items WHERE owner = ? AND itemname = ?"
);
stmt.setString(1, userName);
stmt.setString(2, itemName);
```

```php
// PHP (safe)
$stmt = $pdo->prepare("SELECT * FROM messages WHERE MessageID = ?");
$stmt->execute([$id]);
```

### ✔️ 2. Use ORM Frameworks
Frameworks like **Hibernate**, **Entity Framework**, or **Django ORM** abstract raw SQL and handle escaping automatically.

### ✔️ 3. Input Validation
- Use an **allowlist** approach — only accept expected characters
- For numeric IDs: `$id = intval($_COOKIE["mid"]);`
- Reject or sanitize unexpected input early

### ✔️ 4. Principle of Least Privilege
- Database users should only have the **minimum required permissions**
- Avoid running apps as DB admin
- Separate read/write accounts where possible

### ✔️ 5. Error Handling
- **Never expose** SQL error messages to end users
- Log errors internally, show generic messages publicly
- Detailed errors help attackers refine their payloads

### ✔️ 6. Web Application Firewall (WAF)
- Acts as a supplementary defense layer
- Not a substitute for proper parameterization
- Effectiveness: **Moderate** — can be bypassed

---

## 🔍 Detection Methods

| Method | Effectiveness |
|--------|--------------|
| Automated Static Analysis (AST) | High |
| Manual Code Review | High |
| Dynamic Analysis / Fuzzing | Moderate |
| Database / Web App Scanners | High |
| Architecture / Design Review | High |

---

## 📚 Real-World CVE Examples

| CVE | Description |
|-----|-------------|
| CVE-2024-6847 | SQL injection in AI chatbot via conversation message |
| CVE-2025-26794 | SQL injection in email agent via SQLite integration |
| CVE-2021-42258 | Time & billing software — exploited in the wild (CISA KEV) |
| CVE-2021-27101 | File-transfer system via crafted Host header (CISA KEV) |
| CVE-2020-12271 | Firewall product admin interface (CISA KEV) |
| CVE-2007-6602 | SQL injection via username field |

---

## 🧩 Key Takeaways

- **Parameterized queries are the gold standard** — not allowlists, not escaping, not stored procedures alone
- Stored procedures can still be vulnerable if they concatenate strings internally
- Allowlists help but are insufficient on their own (e.g., apostrophes in names like `O'Reilly`)
- Client-side validation is **never sufficient** — always validate server-side
- Error messages are an attacker's best friend — keep them vague externally

---

## 🔗 References

- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [CISA Secure by Design Alert — Eliminating SQL Injection](https://www.cisa.gov/resources-tools/resources/secure-design-alert-eliminating-sql-injection-vulnerabilities-software)
- [CWE-89 Official Entry](https://cwe.mitre.org/data/definitions/89.html)
- [CAPEC-66: SQL Injection](https://capec.mitre.org/data/definitions/66.html)

---

*Based on CWE-89 from the MITRE Common Weakness Enumeration. Part of the CWE Top 25 Most Dangerous Software Weaknesses (2025).*
