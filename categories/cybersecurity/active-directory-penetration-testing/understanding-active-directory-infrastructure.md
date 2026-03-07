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

Active Directory also includes:

* A set of rules, **the schema**, that defines the classes of objects and attributes contained\
  in the directory, the constraints and limits on instances of these objects, and the\
  format of their names.
* A **global catalog** that contains information about every object in the directory. This\
  allows users and administrators to find directory information regardless of which\
  domain in the directory actually contains the data.
* A **query and index mechanism**, so that objects and their properties can be published\
  and found by network users or applications. For more information about querying\
  the directory.
* A **replication** service that distributes directory data across a network. All domain\
  controllers in a domain participate in replication and contain a complete copy of all\
  directory information for their domain. Any change to directory data is replicated to\
  all domain controllers in the domain.

#### Key Concepts

* **Domains, domain controllers, forests, and trust relationships** form the core structure of an Active Directory environment.
* **Domains** organize users, computers, and resources within a defined security boundary, while **domain controllers** handle authentication and policy enforcement.
* **Forests** connect multiple domains under a shared configuration, and **trust relationships** enable controlled access to resources between domains.
* An **Organizational Unit (OU)** in **Active Directory** is a container used to organize users, computers, groups, and other objects within a domain. It helps administrators manage resources more easily and apply **Group Policy** settings to specific groups of objects.

## Active Directory Domain Service (AD DS)

**Active Directory Domain Services** is the core component of **Active Directory** that provides **directory services for managing users, computers, and other resources in a network**. It works as a centralized system where all information about network objects—such as user accounts, groups, computers, and security policies—is stored and organized in a directory database.

This service runs on **Windows Server** and allows administrators to control authentication and authorization across the network. When a user logs into a domain computer, AD DS verifies the user’s credentials and determines what resources they are allowed to access. It also enables administrators to manage permissions, apply group policies, and organize network objects in a structured hierarchy using domains, organizational units, and forests.

## Active Directory Structure&#x20;

Active Directory's main components, which we use to design the hierarchy and to optimize network traffic, are its logical structure and its physical structure. The logical structure, which simply organizes network resources, consists of OUs, domains, trees, and forests. The logical structure helps you design a network hierarchy that suits your organizational needs. We use the physical structure, which consists of sites and domain controllers, to manage and optimize network traffic by customizing the network configuration.

### Physical Structure of ADDS

The physical structure of Active Directory helps to manage the communication between servers with respect to the directory. The two physical elements of Active Directory are **domain controllers** and **sites**.

#### Domain Controller (DC)

A **Domain Controller** is a server that runs **Active Directory Domain Services** and is responsible for managing authentication, authorization, and directory data within a domain of **Active Directory**. It stores the Active Directory database and handles requests such as user logins, permission checks, and access to network resources. DC host other services that are complementary to AD DS as well. Those are:

* **Kerberos Key Distribution Center (KDC):** The KDC verifies and encrypts kerberos tickets that AD DS uses for authentication.
* **NetLogon:** NetLogon is the authentication communication service.
* **Windows Time (W32time):** Kerberos requires all computer times to be in sync.
* **Intersite Messaging (ISMServ):** Intersite messaging allows DCs to communicate with each other for replication and site-routing.

When a user attempts to log in to a domain computer, the Domain Controller verifies the username and password and determines whether the user is allowed to access the requested resources. It also enforces security policies and replicates directory information with other domain controllers to keep the directory consistent across the network.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### Sites

In **Active Directory**, a **site** represents the **physical structure of a network**. It is used to group domain controllers and other network resources that are located in the same physical location or connected by a high-speed network.

A site is mainly created to manage **network traffic and replication between domain controllers**. When multiple offices exist in different geographic locations, administrators define sites so that **authentication requests and directory replication occur efficiently** within the same local network before communicating with remote locations. This helps reduce bandwidth usage across slower WAN links.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Logical Structure of ADDS

The logical parts of Active Directory include Forests, Trees, Domains, OUs and Global Catalogs.

#### Domain

The basic organizational structure of the Windows Server OS networking model is the domain. A domain represents an administrative boundary. The computers, users, and other objects within a domain share a common security database.

#### Tree

Multiple domains are organized into a hierarchical structure called a tree. Actually, even if you have only one domain in your organization, you still have a tree. The first domain you create in a tree is called the root domain. The next domain that you add becomes a child domain of that root. This\
expandability of domains makes it possible to have many domains in a tree.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### Forest

A forest is a group of one or more domain trees that do not form a contiguous namespace but may share a common schema and global catalog. There is always at least one forest on a network, and it is created when the first Active Directory-enabled computer (domain controller) on a network is installed. This first domain in a forest, called the forest root domain, is special because it holds the schema and controls domain naming for the entire forest. It cannot be removed from the forest without removing the entire forest itself. Also, no other domain can ever be created above the forest root domain in the forest domain hierarchy. A forest is the outermost boundary of Active Directory; the directory cannot be larger than the forest. The following figure shows an example of a forest with two trees. In this figure, Trees in a forest share the same schema, but not the same namespace.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Organizational Unit (OU)

Organizational Units (OUs) provide a way to create administrative boundaries within a domain. Primarily, this allows you to delegate administrative tasks within the domain. OUs serve as containers into which the resources of a domain can be placed. You can then assign administrative permissions on the OU itself. We can even nest OUs (create OUs inside other OUs) for further control.
