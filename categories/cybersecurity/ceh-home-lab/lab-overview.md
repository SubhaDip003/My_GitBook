---
description: Detail overview of the lab.
icon: bars
cover: ../../../.gitbook/assets/LAB OVERVIEW MAIN.png
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

# LAB OVERVIEW

## Lab Setup Requirements

* **Processor** - Intel Core-i5 (Quad Core or higher)
* **HDD/SSD** - 200 GB of Free Disk Space
* **RAM** - 8 GB or More of RAM size
* **NIC** - 1 (Network interface card)
* **Wireless** - 2 Wireless Network Adaptors (PCI or USB)
* **Monitor** - X
* Keyboard/Mouse - X

## Lab Prerequisites



## Available Machines

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

**The detailed information on the machines is given below:**

## 🪟Windows11

* **Username:** Admin
* **Password:** Pa\$$w0rd
* **Machine Name:** Windows11

**USER ACCOUNTS**

| USERNAMES | PASSWORDS | ACCOUNT TYPES |
| --------- | --------- | ------------- |
| Martin    | apple     | Standard      |
| Jason     | qwerty    | Administrator |
| Shiela    | test      | Standard      |

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.11
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **DNS:** 8.8.8.8

Additional things to remember...

* You can find CEH Tools at E:\CEH-Tools location.
* Ensure that the Windows Defender Firewall is off.
* Ensure that Remote Desktop Connection is enable.
* Ensure that the Password Complexity is not set.

***

## 🪟Windows Server 2022

* **Username:** CEH\Administrator
* **Password:** Pa\$$w0rd
* **Machine Name:** Server2022
* **Domain Name:** CEH.com

**ACTIVE DIRECTORY USERS**

| USERNAMES | PASSWORDS | ACCOUNT TYPES |
| --------- | --------- | ------------- |
| Jason M.  | qwerty    | Administrator |
| Martin J. | apple     | Standard      |
| Shiela D. | test      | Standard      |
| Mark      | cupcake   | Standard      |
| SQL\_srv  | batman    | Administrator |
| Joshua    | cupcake   | Standard      |
| DC-Admin  | advance!  | Administrator |

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.22
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **DNS:** 8.8.8.8

Additional things to remember...

* You can find CEH Tools in Z:\ Drive
* Ensure that the Windows Defender Firewall is off.
* Ensure that Remote Desktop Connection is enable.
* Ensure that the Password Complexity is not set.
* Configure WampServer and Wordpress Website:

http://localhost:8080/DVWA/login.php with credentials admin:password

***

## 🪟Windows Server 2019

* **Username:** Administrator
* **Password:** Pa\$$w0rd
* **Machine Name:** Server2019

**USER ACCOUNTS**

| USERNAMES | PASSWORDS | ACCOUNT TYPES |
| --------- | --------- | ------------- |
| Martin    | apple     | Standard      |
| Jason     | qwerty    | Administrator |
| Shiela    | test      | Standard      |

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.19
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **DNS:** 8.8.8.8

Additional things to remember...

* You can find CEH Tools in Z:\ Drive
* Ensure that the Windows Defender Firewall is off.
* Ensure that Remote Desktop Connection is enable.
* Ensure that the Password Complexity is not set.
* WEBSITES
  * http://www.goodshopping.com
  * htt://www.moviescope.com

***

## 🪟Windows 11 (AD)

* **Username:** Mark
* **Password:** cupcake
* **Machine Name:** Windows11

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.40
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **Preferred DNS:** 10.10.1.22

Additional things to remember...

* Ensure that the Windows Defender Firewall is off.
* Ensure that Remote Desktop Connection is enable.
* Ensure that the Password Complexity is not set.

***

## 🪟Windows Server 2019 (AD)

* **Username:** SQL\_srv
* **Password:** batman
* **Machine Name:** Server2019

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.30
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **Preferred DNS:** 10.10.1.22

Additional things to remember...

* Ensure that the Windows Defender Firewall is off.
* Ensure that Remote Desktop Connection is enable.
* Ensure that the Password Complexity is not set.

***

## 🐧Parrot Security

* **Username:** attacker
* **Password:** toor
* **Machine Name:** parrot

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.13
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **DNS:** 8.8.8.8

***

## 🐧Ubuntu

* **Username:** Ubuntu
* **Password:** toor
* **Machine Name:** ubuntu

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.9
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **DNS:** 8.8.8.8

***

## 📱Android

* **Username:** nil
* **Password:** nil

**NETWORK SETTINGS**

* **IP Address:** 10.10.1.14
* **Subnet Mask:** 255.255.255.0
* **Default Gateway:** 10.10.1.2
* **DNS:** 8.8.8.8

***

## Our Lab Structure

In our home lab, we use:

1. Parrot OS
2. Windows 11
3. Windows Server 2022 AD
4. Windows Server 2019 + AD
5. Ubuntu
6. Android

That means we use one common windows server 2019 machine for both as a normal machine where web servers are running and AD Client where SQL server is running.

This is help us to salf our system resource.

***

## Integrate ShellGPT in the Parrot Security Machine

The Certified Ethical Hacker (CEH) labs use an innovative Al tool called ShellGPT. Powered by advanced natural language processing, ShellGPT enables users to interact with command-line interfaces intuitively. Whether probing network vulnerabilities or executing penetration tests, ShellGPT responds to commands in real-time, streamlining tasks and enhancing productivity. Its adaptive capabilities facilitate seamless navigation through complex systems, empowering learners to explore diverse cybersecurity scenarios with ease. Al-powered ethical hacking significantly enhances both efficiency and precision in ethical hacking engagements.

The steps to integrate ShellGPT in the Parrot Security machine are given below:

* **Caution:** To complete the steps below, you must sign up for OpenAl services and add a payment method for the OpenAl API keys for ShellGPT. You will be required to provide your credit card information.
* **Notice:** Third-Party Connected Apps and Paid Services

**Introduction**\
The guiding principle of EC-Council Education Content and Labs is simple: We want to provide a safe environment to learn, practice, and assess cybersecurity skills that directly translate to job-ready skills. We believe in providing our students, educator partners, and professionals with real software and live experiences that behave like those in the "real world" because they are the actual tools and services used in the field, not simulations.

**Third-Party and Internet-Connected Resources**

To provide the most realistic learning experience, EC-Council may employ connections to live networks and services through its labs and lab platforms. While many software and service providers provide education use licenses, trials, or are Open Source, some require a live account with setup requirements and agreements you must review and consent to in order to participate in and/or complete the exercises in our Labs.

**Subscription Fees and/or License Fees**

While EC-Council will never ask you for payment details or sensitive information during our course of learning, some of our Live Labs use paid OpenAI API keys. To use these, you will need to add credit card information for a payment method and buy API credits. Similarly, some of our Live Labs connect to third-party services such as Amazon AWS, Google Cloud Platform, Microsoft Azure, etc. Many of these service providers allow you to sign up for free-tier usage, Education Accounts, or provide promotional credits for trials. However, be aware that signing up for third-party services may incur expenses related to the use of their services. Our labs using public cloud services can be completed without any additional charges unless explicitly marked, but you are ultimately responsible for any services you may intentionally or unintentionally sign up for when connected to those platforms.

**Alternatives**

If you are unable to buy OpenAI API credits, secure an Education/Student cloud account or are not comfortable creating a live account in these public cloud services, EC-Council provides comprehensive Lab Guides that include complete instruction sets with screenshots (images) of each step. Though you won't be creating an account or performing the live configurations with this method, you will still receive adequate exposure to the platforms, configuration steps, techniques, and knowledge elements required for you to prepare for certification or grasp the knowledge conveyed in the program.



1. Visit the OpenAl platform (**https://platform.openai.com**) and sign up or log in to your existing account.
2. Once logged in, click the profile icon at the top-right, click **Your profile** option, and\
   navigate to the **Billing** section.
3. Under **Billing** section, go to **Payment methods** and click Add payment method to add a payment method.
4. Go to **Overview** and click **Add to credit balance**.
5. Enter the minimum amount, i.e., **$5** for **Amount to add**, and click **continue**. Please refer **https://openai.com/api/pricing/** for API pricing details.
6. Click **Confirm payment** and complete the transaction to add the credit balance.
7. Now, switch to the **Parrot Security** machine and login with **attacker/toor**.
8. Open a **Terminal** window, type **chromium**, and press **Enter** to launch the **Chromium** web browser.

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

9. The Chromium window appears, go to **https://platform.openai.com/api-keys.**
10. An **OpenAl Platform** webpage appears, click on **Login** and login to the OpenAl account that you created at the start of this lab.
11. You will be redirected to **Project API Keys** window, click on + **Create new secret key** button to create a new key.

<figure><img src="../../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

12. A **Create new secret key** window appears, provide a name for the secret key (here, **CEH**) and leave all the settings as default, as shown in the screenshot and click on **Create secret key** button.

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

13. **Save your key** pop-up appears, click **Copy** button to copy the key.\
    **Note:** Save this key in a notepad. We will be using this key in each module for setting up ShellGPT.

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

14. Now, open a new terminal with superuser privileges by executing the **sudo su** command (When prompted, enter the password **toor**).
15. Run **sgpt** command to launch ShellGPT tool.

<figure><img src="../../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

16. You will be prompted to enter your OpenAI API key, paste the copied API key in the terminal window and press **Enter**.\
    **Note**: The secret key you have entered will not be visible.
17. If the API key integration is successful, you should receive a message, as shown in the screenshot

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

18. We have successfully integrated ShellGPT API. We will use ShellGPT in the upcoming lab tasks.
19. Close all the open windows.
