---
description: Explore and Learn
icon: server
cover: >-
  ../../../.gitbook/assets/cyber-security-expert-working-with-technology-neon-lights.jpg
coverY: 0
layout:
  width: wide
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: false
  pagination:
    visible: true
  metadata:
    visible: false
  tags:
    visible: true
---

# Machines - HTB

HackTheBox Machines walkthroughs  — clear, practical guides with commands, screenshots, and analysis. Start learning by doing: read, replicate, and level up your offensive security skills.

## Walkthroughs and Techniques

<table data-view="cards"><thead><tr><th></th><th data-hidden data-card-cover data-type="image">Cover image</th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td>Environment is a medium Linux machine where an <code>--env</code> login bypass (CVE-2024-52301) and a management-dashboard file-upload flaw (CVE-2024-2154) lead to a PHP webshell. From the compromised host you recover GPG keys and passwords for SSH, then abuse preserved <code>BASH_ENV</code> in a sudoed script to run commands as root.</td><td><a href="../../../.gitbook/assets/Environment.png">Environment.png</a></td><td><a href="https://medium.com/@subhadipsardar866/hack-the-box-environment-wolkthrough-6a9af5281de0">https://medium.com/@subhadipsardar866/hack-the-box-environment-wolkthrough-6a9af5281de0</a></td></tr><tr><td>Fluffy is an easy Windows box built around an assumed-breach scenario: you start with credentials for a low-privileged user, then exploit CVE-2025-24071 to recover another low-privileged account’s credentials. Enumeration reveals ACLs protecting the <code>winrm_svc</code> and <code>ca_svc</code> service accounts, allowing you to authenticate over WinRM as <code>winrm_svc</code>. Finally, by abusing the Active Directory Certificate Service (an ESC16-style abuse) with the <code>ca_svc</code> account, you can obtain credentials that lead to full Administrator access.</td><td><a href="../../../.gitbook/assets/Fluffy.png">Fluffy.png</a></td><td><a href="https://medium.com/@subhadipsardar866/hack-the-box-fluffy-machine-walkthrough-fa005e25af31">https://medium.com/@subhadipsardar866/hack-the-box-fluffy-machine-walkthrough-fa005e25af31</a></td></tr><tr><td>Puppy (Medium) boils down to an SMB-driven domain pivot. Using levi.james’s credentials we enumerate the domain, abuse Levi’s GenericWrite to add him to the Developers group, and access the DEV share to recover and crack a KeePass backup for credentials. A password-spray hits Ant.Edwards; Ant’s GenericAll over Adam.Silver lets us reset Adam’s password and log in via WinRM, then a website backup reveals Steph.cooper’s password which decrypts DPAPI secrets to expose Steph.cooper_adm and obtain final administrator WinRM access.</td><td><a href="../../../.gitbook/assets/Puppy.png">Puppy.png</a></td><td><a href="https://medium.com/@subhadipsardar866/hack-the-box-puppy-machine-walk-through-b8ddab9d701c">https://medium.com/@subhadipsardar866/hack-the-box-puppy-machine-walk-through-b8ddab9d701c</a></td></tr><tr><td>Certificate is a hard Windows Active Directory machine that starts with an E-learning platform. An initial null-byte file-upload flaw allowed a PHP web shell and initial access as <strong>xamppuser</strong>; database credentials led to <strong>Sara.B</strong>, and a captured network trace exposed <strong>Lion.SK</strong>’s account. Using those credentials the attacker enumerated AD Certificate Services, abused a vulnerable template to request a certificate for <strong>Ryan.K</strong>, then leveraged that certificate and <strong>SeManageVolumePrivilege</strong> to obtain a shell as <strong>NT AUTHORITY\NETWORK SERVICE</strong>. With <strong>SeImpersonatePrivilege</strong> they escalated to <strong>NT AUTHORITY\SYSTEM</strong>, dumped AD/registry hives to extract the Administrator’s NTLM hash, and ultimately logged in as Administrator.</td><td><a href="../../../.gitbook/assets/Certificate.png">Certificate.png</a></td><td><a href="https://medium.com/@subhadipsardar866/hack-the-box-certificate-machine-walk-through-fa6d5b22c4b8">https://medium.com/@subhadipsardar866/hack-the-box-certificate-machine-walk-through-fa6d5b22c4b8</a></td></tr><tr><td><code>TombWatcher</code> is a medium difficulty machine, multiple <strong>DACL abuse</strong> chaining leads to shell as <code>john</code>, <code>john</code> is able to create shadow credential for <code>cert_admin</code> after restoring it from AD recycle bin, exploiting <strong>ADCS ESC15</strong> gains us WINRM access to domain controller as <code>Administrator</code>.</td><td><a href="../../../.gitbook/assets/TombWatcher.png">TombWatcher.png</a></td><td><a href="https://medium.com/@subhadipsardar866/hack-the-box-tombwatcher-season-8-machine-walk-through-0a3a4a730289">https://medium.com/@subhadipsardar866/hack-the-box-tombwatcher-season-8-machine-walk-through-0a3a4a730289</a></td></tr><tr><td>Outbound is an easy Linux machine that begins with assumed-breach credentials for a Roundcube instance. By enumerating the version and exploiting CVE-2025-49113 (authenticated PHP deserialization RCE), we gain initial access and extract a database session for the user Jacob. Decoding and decrypting the stored credential gives us Jacob’s Roundcube login, where we find his updated system password and a notice granting him sudo access to the <code>below</code> utility. The <code>below</code> tool is vulnerable to CVE-2025-27591, allowing a symlink attack on log files. By linking <code>/etc/passwd</code> to the log and injecting a payload, we create a root-privileged user and escalate to full system compromise.</td><td><a href="../../../.gitbook/assets/2025-11-16_12-25.png">2025-11-16_12-25.png</a></td><td><a href="https://medium.com/@subhadipsardar866/hack-the-box-outbound-machine-walk-through-b245ab9b467b">https://medium.com/@subhadipsardar866/hack-the-box-outbound-machine-walk-through-b245ab9b467b</a></td></tr></tbody></table>
