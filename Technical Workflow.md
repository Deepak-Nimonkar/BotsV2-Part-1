## Technical Attack Sequence

[45.77.65.211] ---> SQL Injection (updatexml) ---> Database Dump (mybb_users)
│
[71.39.18.125] <--- CSRF / XSS Victim Browser --- Silently Executes POST
│
▼
Creates Backdoor Admin: kIagerfield
│
[136.0.2.138] ---> Authenticates via kIagerfield ---> Modifies Config (gid=10)

### 1. Reconnaissance & Database Extraction (SQLi)
* **Attacker IP:** `45.77.65.211`
* **Target Endpoint:** `/member.php`
* **Vector:** Error-based SQL Injection using `updatexml()` functions.
* **Impact:** Extracting system details and credentials from the `mybb_users` table.

### 2. Backdoor Account Creation via Cross-Site Request Forgery (CSRF)
* **Victim IP:** `71.39.18.125` (Legitimate Administrator: Kevin Lagerfield)
* **Timestamp:** `2017-08-16 20:49:18 UTC`
* **Mechanism:** The attacker tricked Kevin’s browser into issuing an authenticated `POST` request to `/admin/index.php` while Kevin was logged into the Admin Control Panel.
* **Artifact:** The request carried Kevin's valid session cookie and anti-CSRF token (`my_post_key=1bc3eab741900ab25c98eee86bf20feb`).
* **Result:** Created rogue admin account **`kIagerfield`** (homograph attack using a capital `I` instead of an `l`) with password `beer_lulz` and `usergroup=4` (Admin).

##http

POST /admin/index.php HTTP/1.1
Host: [www.brewertalk.com](https://www.brewertalk.com)
Cookie: [Kevin's Session Cookies]

my_post_key=1bc3eab741900ab25c98eee86bf20feb&username=kIagerfield&password=beer_lulz&confirm_password=beer_lulz&email=kIagerfield@froth.ly&usergroup=4


##3. Unauthorized Administrative Access

Attacker IP: 136.0.2.138
User-Agent: NaenaraBrowser/3.5b4 (Red Star OS browser identifier)
Login Event: 2017-08-16 20:57:15 UTC via /member.php.
Configuration Tampering (20:58:44 UTC): Submitted a modified configuration form (gid=10) changing the avatar upload path (upsetting[avataruploadpath]=./uploads/avatars).
Session Termination: The attacker's web session abruptly ended at 20:58:45.446 UTC.

##Post-Exploitation & Rule-Out Analysis

| Vector | Query / Source | Findings | Status |
| :--- | :--- | :--- | :--- |
| **Web Shell Upload** | `stream:http` (`uri_path="*usercp*" OR uri_path="*uploads*"`) | 0 requests to `usercp.php` or files inside `/uploads/` post-20:58:45. | **Ruled Out** |
| **Host System Calls** | `linux_audit` (`EXECVE`, `SYSCALL`) | No file writes or execution events triggered by `www-data` or `apache2`. | **Ruled Out** |
| **SSH Compromise** | `linux_audit` (`182.48.234.27`) | `CRYPTO_KEY_USER` succeeded, but 100% of PAM authentication (`USER_AUTH`/`USER_LOGIN`) failed. | **Ruled Out (Noise)** |

