---
title: Domain Name System (DNS) DeepDive - Registry, Registrar
date: 2025-06-28T11:46:25.268Z
draft: false
layout: PostDefault
summary:
tags:
  - DNS
  - Registry
  - Registrar
lastmod: 2025-07-01T21:41:45+09:00
category: develop
series: DNS-DeepDive
seriesOrder: 2
withAI: true
---

> The Internet is for Everyone

## Intro

In the last article, we learned about the background and system of DNS and domain names. In this article, we're going to learn about domain names and the registration and management structure that keeps DNS running smoothly.

## What is a registry?

The Internet is built on decentralized management, with no single entity centralizing the entire Internet.
The exception to this is identifiers like IP addresses and domain names to identify hosts connected to the Internet, which require centralized management. The organization that does this is called a **registry**.

> [!note] Registries and registry operators
> The term "registry" is used in two senses.
>
> 1. a registry operator that assigns identifiers and manages registrations.
> 2. a registry database, which is the register handled by a registry operator.
>
> > In this article, the "registry operator" in 1. is referred to as the registry.

### Differences in management between IP addresses and domain names

IP addresses and domain names are handled differently for allocation and registration management due to their respective characteristics.
Because IP addresses share a finite resource across the Internet, they need to be managed **consistently** on a global scale.
To accomplish this, IP address registries are responsible for setting rules or coordinating between registries.
Domain names, on the other hand, have a single namespace that is partitioned across TLDs, and **have the flexibility to operate according to the specifics of each TLD**.

### The role of registries

In order to use a domain name, you must apply to a registry for registration. The registry then reviews and verifies that the application meets the registration requirements and registers it in its database.

The main roles of registries are

1. manage the operation of the registry database
   - It operates a registry database that stores information such as names of individuals, organizations, and contact information required to register a domain name.
2. Establishing registration rules in accordance with policy
   - The registry establishes the **policy** of the domain names it registers and manages, and establishes **registration rules** to realize the policy and notifies users.
3. Receiving registration applications
   - The registry receives applications for registration of domain names from registrants. It examines the applied domain names according to the rules and registers the information received in the registry database.
4. Provide Whois services
   - The registry provides information on domain names it manages as a `Whois` service.
5. Operate nameservers
   - Manages and operates **Nameservers** to make the domains it manages available on the Internet.

### Relationship between registries and TLDs

A registry exists for each Top Level Domain (TLD), and it manages each registry TLD and establishes management policies or registration rules.

There are two main types of TLDs.

1. Country Code Top Level Domain (ccTLD), a domain assigned to each country or region.
   - ex) `.kr`, `.jp`, `.cn`
2. Generic Top Level Domain (gTLD), a domain that is independent of country or region.
   - ex) `.com`, `.net`, `.org`

## Registries and the Registrar Model and the Role of Registrars

In order to use a domain name, it must be registered in the registry database of the registry that manages it.

Today, the major TLDs (.com/.net/.kr) use the **registry⋅registrar model** as their registration structure.

### Registry⋅registrar model

The registry⋅registrar model separates the management of domain name registrations into two roles, as follows

![[image-107.webp]]

The reason for this separation of roles is to ensure that the names of the domains being registered are unique, while allowing for diversity in pricing and services. This is why there is only one registry but multiple registrars for a single TLD.

![[image-110.webp]]

Each registrar can offer different services to registrants. Registrars can set their own services and pricing, and registrants can choose the registrar that's right for them.

In Korea, Gabia and HostingKR are examples of registrar services.

### Role of a registrar

1. Receive registration applications from registrants
2. Request registration in the registry database
3. providing WHOIS services
4. managing registrant information

## Registering domain names

Let's take a look at the flow of registering a domain name in the registry⋅registrar model.

![[image-108.webp]]

**Applicant** checks through the `whois` service if the domain name is available for registration and selects a provider to apply for the domain name. They compare prices and services and choose the one they like. Next, they prepare the information they need to submit to the **registrar**.
Once this is done, they apply to the registrar to register the domain name.

The **registrar** will verify your application and submit it to the **registry** for registration in the registry database.

The registry verifies that the application received from the registrar is valid and, if it meets the registration requirements, registers the information in its database. It then notifies the registrar that the registration is complete. Upon receiving the notification, the registrar notifies the user that the domain name is now registered.

## Using your domain name

Above, the user has registered a domain name. However, just registering a domain name doesn't make it usable. To make the website with the domain name accessible with a web browser, some preparation is required.

First, you need to register a **name server** that manages the information to make that domain name available on the internet, known as DNS information. You can build your own name server or use an external service like cloudflare. The name server must also be accessible from anywhere on the internet.

Second, set the information for your domain name on the name server you initially set up. For example, if your registered domain name is `jinmu.me`, set the information that `jinmu.me` has an IP address of `76.76.21.21` on the name server.

The final step is to make sure that the name server can answer questions from the Internet based on the information you've registered. For example, it should be able to answer the question, "What is the IP address of `jinmu.me`?" with the answer `76.76.21.21`.

### What the registry does

The registry registers name server information in the registry database and sets that information on the name servers it manages. It sets the name server information that contains the information for `jinmu.me` to the name servers in the ME zone that the registry manages. The parent (me) sets the name server information of the child (jinmu.me), and the relationship between the registry and the registrant becomes a parent-child relationship as described in DNS.

After completing the above procedure, you can actually build a web server and see if you can actually access a website with that domain name (jinmu.me).

![[image-112.webp]]

## Outro

In this article, we took a closer look at the registration and management structures that are necessary for domain names to function smoothly. From the role of the registry in centralizing and managing Internet identifiers to the function of the registrar in brokering domain registrations for users, we saw how the registry-registrar model, which is adopted by most TLDs today, enables reliable management of domain names and competition in service and price, and how the actual domain registration process works. Lastly, we also pointed out that there's more to it than just registering a domain - you need to set up a name server and have it registered with its parent domain before it can be used on the internet.

In the next post, we'll discuss how our computers resolve domain names once they're registered and set up, a process called DNS name resolution, and why delegation is a key principle in making this huge system work smoothly.

Thank you.
