---
description: Step-by-step guide to setting up VMware Network Adaptor.
layout:
  width: default
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
    visible: true
  tags:
    visible: true
---

# VMware Network Adaptor Settings

Open VMWare Workstation --> Click on Edit --> Click on Virtual Network Editor.

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

On the Next page click on "Change Setting" option.

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

User Account Control pop-up will be open click on Yes option.

Then click on  VMnet8 -> Select NAT -> Set Subnet IP: 192.168.10.0 and Subnet Mask: 255.255.255.0

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

On the Next, Same Select VMnet1 -> Select Host-Only -> Set Subnet IP: 10.10.10.0 and Subnet Mask: 255.255.255.0.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Next you can also select another VMNet (VMnet0) from Add Network option and configure as a Bridged Adapter.

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

After successfully configure everything, click on Apply and then OK.

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>
