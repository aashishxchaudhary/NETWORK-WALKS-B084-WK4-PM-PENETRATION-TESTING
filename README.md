# Week 04 — Web Application Security Assessment

**Guided Black-Box Penetration Testing | Networkwalks Cybersecurity Program — Batch B084**

![Penetration Testing](https://img.shields.io/badge/Domain-Penetration%20Testing-red?style=for-the-badge)
![Kali Linux](https://img.shields.io/badge/Platform-Kali%20Linux-blue?style=for-the-badge)
![Web Security](https://img.shields.io/badge/Focus-Web%20Security-purple?style=for-the-badge)
![Networkwalks](https://img.shields.io/badge/Program-Networkwalks%20B084-black?style=for-the-badge)

---

## 1. Project Overview

This project documents a guided black-box penetration testing exercise conducted against **Mediroza General Hospital**, a practice web application provided for the Networkwalks Cybersecurity Program.

The assessment focused on identifying web application vulnerabilities, evaluating document password security, examining file metadata, and investigating publicly accessible server resources.

The exercise demonstrated how multiple security weaknesses can combine to create a chain of exposure affecting authentication mechanisms, confidential documents, and internal organizational information.

### Assessment Highlights

- SQL Injection and authentication bypass testing.
- Offline password security assessment of encrypted PDF documents.
- PDF metadata inspection and information discovery.
- Identification of exposed server directories.
- Database backup exposure assessment.
- Vulnerability classification and remediation recommendations.

## 2. Assessment Information

| Field | Details |
|---|---|
| Researcher | Aashish Kumar Chaudhary |
| Program | Networkwalks Cybersecurity Program |
| Batch | B084 |
| Project | Week 04 — Web Application Security Assessment |
| Target | `medirozahospital.com` |
| Environment | Kali Linux |
| Assessment Type | Black-Box Penetration Testing |
| Assessment Date | 23 November 2026 |
| Primary Focus | Web Application & Information Disclosure |
| Status | Completed |

## 3. Legal Disclaimer

This project was performed as an educational cybersecurity exercise against a designated practice target provided as part of the Networkwalks Cybersecurity Program.

All testing activities documented here relate to the authorized lab exercise.

This repository is intended strictly for educational and defensive security research. It does not encourage unauthorized access, exploitation, or disclosure of real personal information.

Screenshots and evidence published in this repository should be appropriately sanitized to prevent exposure of sensitive information.

## 4. Executive Summary

The assessment identified three major security weaknesses across the practice application and its publicly accessible resources.

| ID | Security Finding | Severity |
|---|---|---|
| VULN-01 | SQL Injection — Authentication Bypass | Critical |
| VULN-02 | Weak PDF Password Protection | High |
| VULN-03 | Exposed Database Backup | Critical |

The exercise demonstrated a potential attack chain involving:

1. Authentication bypass through SQL Injection.
2. Retrieval of password-protected PDF documents.
3. Offline assessment of weak document passwords.
4. Discovery of sensitive server information through PDF metadata.
5. Identification of an exposed database backup.

These findings demonstrate the importance of secure authentication, appropriate document protection, metadata sanitization, and server-side access controls.

## 5. Scope & Methodology

### Scope

| Component | Assessment Activity |
|---|---|
| Patient Portal | Authentication security testing |
| Login Interface | SQL Injection testing |
| PDF Documents | Password security assessment |
| Document Metadata | Information disclosure analysis |
| Public Directories | Directory exposure assessment |
| Database Backup | Exposure and content review |

### Testing Methodology

**Phase 1 — Reconnaissance**
- Website navigation and endpoint identification.
- Login interface inspection.
- HTML source examination.

**Phase 2 — Authentication Testing**
- SQL Injection testing.
- Database error analysis.
- Authentication bypass validation.

**Phase 3 — Document Security Assessment**
- PDF retrieval and encryption inspection.
- Password hash extraction.
- Offline password auditing.
- Document metadata analysis.

**Phase 4 — Information Disclosure**
- Investigation of metadata references.
- Public directory inspection.
- Database backup exposure assessment.
- Security impact evaluation.

## 6. Technical Findings

### VULN-01 — SQL Injection Authentication Bypass

**Severity:** CRITICAL

**CWE:** CWE-89 — Improper Neutralization of Special Elements Used in an SQL Command

#### Description

During login interface testing, the application was found to exhibit SQL Injection behavior.

A specially crafted input resulted in a database error, indicating that user-controlled input was potentially being incorporated into SQL statements without appropriate parameterization.

Further testing demonstrated an authentication bypass in the practice environment.

#### Technical Evidence

**Observed input pattern:**

```sql
' OR '1'='1
```

**Authentication bypass test:**

```sql
admin'-- -
```

The application subsequently granted access to the patient portal without a valid password.

#### Evidence

![Login Interface](screenshots/M1-Reconnaissance/Login-Page.png)

![SQL Injection Error](screenshots/M1-SQL-Injection/SQL-Error.png)

![Authentication Bypass](screenshots/M1-SQL-Injection/Login-Success.png)

#### Security Impact

- Unauthorized authentication.
- Potential account compromise.
- Unauthorized access to restricted application resources.
- Potential exposure of confidential documents.

#### Remediation

1. Implement parameterized SQL queries and prepared statements.
2. Avoid dynamically concatenating untrusted input into SQL statements.
3. Apply server-side input validation.
4. Use database accounts with least-privilege permissions.
5. Prevent raw database errors from being displayed to end users.
6. Conduct security testing on authentication endpoints.

---

### VULN-02 — Weak PDF Password Protection

**Severity:** HIGH

**CWE:** CWE-521 — Weak Password Requirements

#### Description

Three encrypted PDF documents were retrieved from the practice portal.

A password security assessment was performed using extracted PDF password hashes and offline dictionary-based testing.

John the Ripper initially failed to load the extracted hashes in the tested environment. An alternative Python implementation using the `pikepdf` library was subsequently used to validate document passwords against the authorized lab wordlist.

The exercise demonstrated that weak document passwords can undermine the confidentiality provided by PDF encryption.

#### Tools & Technologies

- `pdf2john`
- John the Ripper
- Python
- pikepdf
- RockYou wordlist

#### Commands Used

**Hash extraction:**

```bash
pdf2john patient_report_1.pdf > hash1.txt
pdf2john patient_report_2.pdf > hash2.txt
pdf2john patient_report_3.pdf > hash3.txt
```

**Password auditing attempt:**

```bash
john --format=PDF \
--wordlist=/usr/share/wordlists/rockyou.txt \
hash1.txt
```

#### Alternative Approach

A Python script using `pikepdf` was used to test candidate passwords against the encrypted documents.

```python
import pikepdf
import sys

pdf_file = sys.argv[1]
wordlist = "/usr/share/wordlists/rockyou.txt"

with open(wordlist, "r", encoding="latin-1") as f:
    for line in f:
        password = line.strip()

        try:
            pdf = pikepdf.open(
                pdf_file,
                password=password
            )

            print("Password successfully validated")
            break

        except pikepdf.PasswordError:
            continue
```

#### Evidence

![PDF Hash Extraction](screenshots/M2-PDF-Analysis/Hash-Extraction.png)

![John the Ripper Result](screenshots/M2-PDF-Analysis/John-Result.png)

![Python PDF Password Audit](screenshots/M2-PDF-Analysis/Python-Audit.png)

![PDF Document Analysis](screenshots/M2-PDF-Analysis/PDF-Analysis.png)

#### Security Impact

- Unauthorized access to protected documents.
- Potential exposure of confidential information.
- Increased risk of information disclosure.
- Weak document-level confidentiality controls.

#### Remediation

1. Use strong, randomly generated document passwords.
2. Avoid predictable passwords and common dictionary words.
3. Apply modern encryption standards.
4. Deliver document passwords through a separate secure communication channel.
5. Restrict access to sensitive documents using appropriate authorization mechanisms.

---

### VULN-03 — Exposed Database Backup

**Severity:** CRITICAL

**CWE:** CWE-552 — Files or Directories Accessible to External Parties

**Related Weakness:** CWE-548 — Exposure of Information Through Directory Listing

#### Description

During PDF metadata inspection, one document contained information referencing a legacy database backup directory.

Further investigation identified a publicly accessible `/old/` directory containing a database backup file.

The exposed backup contained organizational information, including employee records and shareholder information.

This demonstrated a significant information disclosure vulnerability caused by inadequate access restrictions on sensitive server resources.

#### Metadata Investigation

PDF metadata inspection was conducted using ExifTool.

```bash
exiftool patient_report_3_unlocked.pdf
```

The metadata contained a reference to a legacy backup location.

#### Directory Exposure

The following resource was identified during the authorized lab exercise:

```text
/old/
```

The directory exposed a database backup file:

```text
mediroza_db_backup_2019.sql
```

#### Evidence

![PDF Metadata](screenshots/M3-Data-Exposure/PDF-Metadata.png)

![Exposed Directory](screenshots/M3-Data-Exposure/Directory-Listing.png)

![Database Backup](screenshots/M3-Data-Exposure/Database-Backup.png)

#### Data Exposure Assessment

The backup contained two major categories of organizational information.

**Staff Records**
- Employee names.
- Job titles and departments.
- Contact information.
- Employee identification information.
- Salary information.
- Joining dates.

**Shareholder Records**
- Shareholder information.
- Ownership percentages.
- Number of shares.
- Share classifications.

No real personal records should be reproduced in this public repository.

#### Security Impact

- Unauthorized access to internal organizational information.
- Exposure of confidential employee records.
- Potential identity and privacy risks.
- Exposure of financial and ownership information.
- Potential misuse of disclosed organizational data.

#### Remediation

1. Remove database backups from publicly accessible web directories.
2. Store backups outside the web root.
3. Disable directory indexing.
4. Implement strict server-side access controls.
5. Audit publicly accessible directories.
6. Remove unnecessary metadata from downloadable documents.
7. Review server access logs for unauthorized access.
8. Rotate credentials if sensitive authentication material was exposed.

---

## 7. Vulnerability Risk Summary

| ID | Vulnerability | Severity | CWE |
|---|---|---|---|
| VULN-01 | SQL Injection — Authentication Bypass | Critical | CWE-89 |
| VULN-02 | Weak PDF Password Protection | High | CWE-521 |
| VULN-03 | Exposed Database Backup | Critical | CWE-552 |
| VULN-04 | Directory Listing Exposure | High | CWE-548 |

*Severity ratings are qualitative lab assessments based on the described impact and are not independently validated CVSS scores. VULN-03 and VULN-04 represent related but distinct weaknesses.*

## 8. Tools & Technologies

| Tool | Purpose |
|---|---|
| Kali Linux | Penetration testing environment |
| Web Browser | Reconnaissance and application testing |
| Developer Tools | HTML source inspection |
| pdf2john | PDF password hash extraction |
| John the Ripper | Offline password auditing |
| Python | Custom security scripting |
| pikepdf | PDF encryption and password validation |
| ExifTool | Metadata inspection |
| Linux Terminal | Command-line security assessment |

## 9. Overall Security Recommendations

| Security Area | Recommended Control |
|---|---|
| Authentication | Prepared SQL statements |
| Input Handling | Server-side validation |
| Database | Least-privilege access |
| PDF Security | Strong passwords and modern encryption |
| Metadata | Sensitive metadata removal |
| File Storage | Backups outside the web root |
| Web Server | Disable directory listing |
| Monitoring | Security logging and access auditing |

## 10. Skills Demonstrated

This exercise provided practical experience in:

- Web application reconnaissance.
- SQL Injection identification.
- Authentication bypass testing.
- Database error analysis.
- Offline password security auditing.
- PDF encryption analysis.
- Python-based security scripting.
- Document metadata inspection.
- Directory listing identification.
- Information disclosure assessment.
- Vulnerability classification.
- Security remediation documentation.

## 11. Key Learnings

1. Improper SQL query handling can compromise authentication mechanisms.
2. Weak passwords can undermine document encryption.
3. Metadata can unintentionally reveal internal infrastructure information.
4. Publicly accessible backup files can expose sensitive organizational data.
5. Multiple security weaknesses can combine into a serious exposure scenario.
6. Effective penetration testing requires identifying vulnerabilities, documenting their impact, and recommending practical remediation.

## 12. Conclusion

The Week 04 assessment demonstrated how weaknesses in authentication, document security, and server configuration can collectively create significant security risks.

The exercise strengthened my understanding of black-box penetration testing, vulnerability discovery, password security assessment, metadata analysis, and professional security reporting.

It also reinforced an important principle of cybersecurity:

**Security is not determined by a single control. It depends on how effectively multiple layers of protection work together.**

## 13. References

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [MITRE CWE-89](https://cwe.mitre.org/data/definitions/89.html)
- [MITRE CWE-521](https://cwe.mitre.org/data/definitions/521.html)
- [MITRE CWE-552](https://cwe.mitre.org/data/definitions/552.html)
- [MITRE CWE-548](https://cwe.mitre.org/data/definitions/548.html)
- [John the Ripper](https://www.openwall.com/john/)
- [ExifTool](https://exiftool.org/)
- [pikepdf Documentation](https://pikepdf.readthedocs.io/)

---

**Author:** Aashish Kumar Chaudhary  
**Program:** Networkwalks Cybersecurity Program  
**Batch:** B084  
**Project:** Week 04 — Web Application Security Assessment

*Learning. Testing. Securing. Improving.*