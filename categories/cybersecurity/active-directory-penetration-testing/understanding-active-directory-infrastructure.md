---
description: Explore and Learn
icon: lightbulb
layout:
  width: wide
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

# Understanding Active Directory Infrastructure

**Microsoft Active Directory** plays a vital role in managing users, computers, and security within an organization’s network. It acts as a centralized directory that stores important information such as user accounts, group memberships, and security policies. For security professionals and penetration testers, understanding how this infrastructure works is essential for identifying weaknesses and potential attack paths.

An Active Directory environment is built from several core components, including **domains**, **domain controllers**, **forests**, and **trust relationships**. A **domain** is a logical structure that groups users, computers, and other resources under a shared security boundary. **Domain controllers** are servers responsible for running Active Directory services, authenticating users, and applying security policies across the domain.

Multiple domains can be combined into a **forest**, which represents the highest level of the Active Directory hierarchy and shares a common schema and configuration. Within and between these domains, **trust relationships** allow users from one domain to access resources in another, enabling collaboration across different parts of an organization’s network.

#### Key Concepts

* **Domains, domain controllers, forests, and trust relationships** form the core structure of an Active Directory environment.
* **Domains** organize users, computers, and resources within a defined security boundary, while **domain controllers** handle authentication and policy enforcement.
* **Forests** connect multiple domains under a shared configuration, and **trust relationships** enable controlled access to resources between domains.
* An **Organizational Unit (OU)** in **Active Directory** is a container used to organize users, computers, groups, and other objects within a domain. It helps administrators manage resources more easily and apply **Group Policy** settings to specific groups of objects.

## Active Directory Domain Service (AD DS)

**Active Directory Domain Services** is the core component of **Active Directory** that provides **directory services for managing users, computers, and other resources in a network**. It works as a centralized system where all information about network objects—such as user accounts, groups, computers, and security policies—is stored and organized in a directory database.

This service runs on **Windows Server** and allows administrators to control authentication and authorization across the network. When a user logs into a domain computer, AD DS verifies the user’s credentials and determines what resources they are allowed to access. It also enables administrators to manage permissions, apply group policies, and organize network objects in a structured hierarchy using domains, organizational units, and forests.

## Domain in Active Directory

A **Domain** in **Active Directory** is a **logical group of network objects**—such as users, computers, groups, and devices—that share the **same database and security policies** and are managed centrally.

#### Key Points:

* A **domain** acts as a **centralized administrative boundary**.
* It stores all objects in the **Active Directory database**.
* Users in the domain can **log in once and access resources across the network**.
* It is managed by **Active Directory Domain Services** running on **Windows Server**.

#### Example:

If an organization has a domain called:

`company.local`

It may contain:

* Users: `alice`, `bob`
* Computers: `PC-01`, `SERVER-01`
* Groups: `IT_Admins`, `HR_Team`

All these objects are managed under the **same domain policies and authentication system**.

## Domain Controller (DC)

A **Domain Controller** is a server that runs **Active Directory Domain Services** and is responsible for managing authentication, authorization, and directory data within a domain of **Active Directory**. It stores the Active Directory database and handles requests such as user logins, permission checks, and access to network resources. DC host other services that are complementary to AD DS as well. Those are:

* **Kerberos Key Distribution Center (KDC):** The KDC verifies and encrypts kerberos tickets that AD DS uses for authentication.
* **NetLogon:** NetLogon is the authentication communication service.
* **Windows Time (W32time):** Kerberos requires all computer times to be in sync.
* **Intersite Messaging (ISMServ):** Intersite messaging allows DCs to communicate with each other for replication and site-routing.

When a user attempts to log in to a domain computer, the Domain Controller verifies the username and password and determines whether the user is allowed to access the requested resources. It also enforces security policies and replicates directory information with other domain controllers to keep the directory consistent across the network.

