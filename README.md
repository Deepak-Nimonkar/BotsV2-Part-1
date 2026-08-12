# Incident Response Investigation: BOTS v2 MyBB Web Compromise & Admin Takeover

## Overview
This repository documents the end-to-end Digital Forensics and Incident Response (DFIR) investigation of a web compromise on a MyBB forum target (`172.31.4.249`) from the Splunk BOTS v2 dataset. The attack progressed from initial automated reconnaissance and Blind/Error-based SQL Injection to database exfiltration, credential cracking, and administrative control panel takeover.

---

## Attack Lifecycle & Key Findings

### 1. Reconnaissance & Probing (2017-08-11)
* **Source IP:** `45.77.65.211`
* **Activity:** Automated scanning via Nikto/DirBuster targeting sensitive endpoints (`/admin/`, `/archive/`, `/PHPINFO.php`).
* **Time-Based SQLi Testing:** Injected `SLEEP()` payloads into `/search.php` via `keywords` parameter, confirming blind SQL injection via response latency spikes (~12 seconds).

### 2. Database Exfiltration via Error-Based SQL Injection (2017-08-16)
* **Target Endpoint:** `/member.php` (Parameter: `question_id`)
* **Vector:** Error-Based SQL Injection leveraging MySQL `updatexml()` XPath evaluation errors.
* **Impact:** Systematic dumping of 6 user accounts (`LIMIT 0,1` through `LIMIT 5,1`) from `mybb_users`.
* **Signature:** HTTP status code `503 Service Unavailable` returned due to unhandled database errors forced by `updatexml()`.

### 3. Credential Compromise & IP Pivot (2017-08-16 20:57)
* **Secondary Attacker IP:** `136.0.2.138`
* **Compromised User:** `kIagerfield` (Administrator)
* **Cracked Credentials:** `beer_lulz`
* **Access:** Successful authentication to both user portal (`/member.php`) and Admin Control Panel (`/admin/index.php`).

### 4. Configuration Tampering & Web Shell Preparation (2017-08-16 20:58)
* **Target Settings:** Group ID 10 (`gid=10` — User Registration & Avatar Options).
* **Malicious Action:** Modified upload configuration parameters (`upsetting[avataruploadpath]=./uploads/avatars`) to establish pathing for post-exploitation file uploads and potential Remote Code Execution (RCE).

---

## Indicators of Compromise (IoCs)

| Threat Type | Indicator | Context |
| :--- | :--- | :--- |
| **Attacker IP 1** | `45.77.65.211` | Automated scanning, SQLi exfiltration |
| **Attacker IP 2** | `136.0.2.138` | Interactive Admin Panel access & config changes |
| **Compromised Account** | `kIagerfield` | MyBB Administrator |
| **Cracked Password** | `beer_lulz` | Plaintext admin password |
| **Target Endpoints** | `/member.php`, `/admin/index.php` | Vulnerable form submission & Admin Portal |
| **Target Directory** | `./uploads/avatars` | Modified upload path (`gid=10`) |

---


  ```spl
  index=botsv2 sourcetype="stream:http" "updatexml" status=503
  | table _time, src_ip, uri_path, form_data
