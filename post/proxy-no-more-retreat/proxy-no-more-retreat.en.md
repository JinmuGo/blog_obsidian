---
title: Proxies, there's nowhere to hide!
date: 2025-07-12T06:31:40.848Z
draft: false
tags:
  - ReverseProxy
  - ForwardProxy
  - Proxy
  - Network
lastmod: 2025-07-15T17:18:42+09:00
category: develop
withAI: true
bannerOnlyText: true
---

## Intro

I didn't answer the question about Reverse Proxy properly in my [Ausg](https://ausg.me/) 9th interview on July 12, so to celebrate (?), I thought I'd take this opportunity to write a blog post about Proxy that I've been procrastinating on.

> [!note]
> This post is based on a variety of sources, but especially on the Proxies chapter of the [Http definitive guide](https://dl.ebooksworld.ir/books/HTTP.The.Definitive.Guide.Brian.Totty.David.Gourley.OReilly.9781565925090.EBooksWorld.ir.pdf). Also, I'm limiting the proxies discussed here to Web Proxies that use the HTTP/HTTPS protocol.

![Nowhere to retreat anymore!](./proxy-no-more-retreat-1752563332515.webp)

## Proxy

Proxy literally means **'agent'** or **'intermediary'**. In the world of the web, a proxy is an intermediary that sits between two entities, a client and a server, and represents HTTP communication between them. Because of the nature of being between the client and the server, a proxy has a dual nature: it is both a **web server** that receives requests from the client and returns responses, and a **web client** that sends requests to the server and receives responses.
A proxy isn't just a messenger that delivers messages; it's more like an all-powerful control tower that inspects all traffic passing through, guides it to the best route, and even rewrites the content.

## Why Use Proxy?

We've already compared proxies to an "all-purpose control tower," so why do we need one? What value do they provide beyond simply delivering messages, and why are we willing to take on the complexity of the TCP/IP architecture to adopt them?

Importantly, most of the features I'm about to describe apply to both forward and reverse proxies, depending on the direction of the traffic.

### Increased security

- Firewall: A proxy acts as a firewall at the perimeter of your network, managing traffic going out or coming in. It can be a **single point of defense** to block certain kinds of malicious requests and protect your internal network.
- Access control and filtering: Preventing access to certain websites in a school or company is what proxies do. You can filter out inappropriate content, centrally manage access to specific resources, and keep a record of all access.
- Anonymization: Because proxies send requests to the server on behalf of the user, they can strip personally identifiable information such as the user's IP address, browser information, and cookies from the HTTP header or body to enhance privacy and anonymity.

### Performance optimization

- Web Caching: One of the most powerful features of a proxy. By storing a copy of frequently requested content (e.g. logo, CSS file, etc.) on the proxy server, when the same request is made in the future, the cached data can be delivered immediately instead of going to the source server, which is far away, increasing speed and reducing traffic to the server.
- Load Balancing: Large services have multiple web servers running simultaneously. Proxies evenly distribute incoming requests across multiple servers, preventing any particular server from being overloaded and increasing the reliability and efficiency of the entire system.
- Server acceleration (reverse proxy): A proxy that sits directly in front of a web server and disguises itself as the web server is called a **reverse proxy** or **surrogate**. It handles heavy tasks like SSL encryption and static content caching, allowing the actual web server to focus on handling core business logic.

## Content transformation and routing

Proxies can go beyond simple forwarding and can also **transcode** content or **route** it to different destinations on demand.

- Transcoding content: A proxy can change the format of content before delivering it to the client. For example, a high-quality image can be converted to a small JPEG file for mobile devices, or a complex webpage can be converted to simple text for delivery.
- Content routing: You can change the destination of traffic based on the content of the request, such as sending requests starting with "/api" to an application server, "/video" to a media streaming server, and so on.

## Where Do Proxies Go?

Proxies are deployed in various locations on your network to serve different purposes. Where a proxy is located and how it directs traffic determines its role and function. In other words, a proxy's location defines its role and function.

### Different deployment locations for proxy servers

Proxies are strategically placed to control the flow of traffic.

- Egress Proxy: Located at the **exit** of the local network. They are primarily used by businesses or organizations to control the traffic that internal users send to the Internet. They are deployed to reduce internet bandwidth costs and for security purposes such as content filtering to prevent employees from accessing harmful sites and virus protection.
- Access / Ingress Proxy: Located at the **access point** of your Internet Service Provider (ISP), it aggregates requests from a large number of customers. Its main purpose is to cache popular content to improve download speeds for users and reduce internet bandwidth costs.
- Surrogate / Reverse Proxy: It sits at the **end of the network**, right in front of the web server. It acts as if it is the web server and receives all requests first. This greatly improves performance by reducing the load on the actual web server, either by defending against DDos attacks or by caching frequently requested content. A Content Delivery Network (CDN) is a prime example of a reverse proxy.

![[proxy-reverse-forward-a-to-z-1752499264714.webp]]

### Proxy hierarchy

Proxies can stand alone, but they can also form a "hierarchy" of multiple proxies connected like a chain. A request message (HTTP Requeset) can travel through multiple proxies sequentially until it reaches its final destination, the origin server.
In this case, the next proxy in the request's path is called the "parent" and the previous proxy is called the "child".

![[proxy-reverse-forward-a-to-z-1752499376498.webp]]

> [!note] Dynamic Routing
> Rather than simply sending requests to a set parent, a proxy can dynamically forward requests to other proxies or servers based on various conditions.
>
> > Below are some examples of Dynamic Routing.
>
> - Load balancing: A child proxy can select a parent proxy to balance the load based on the current workload level of the parent proxy. There are different algorithms for load balancing, so you may want to look them up if you're curious.
> - Geographic proximity routing: A child proxy can choose a parent proxy that serves the geographic region of the origin server.
> - Protocol/type routing: Child proxies can route to different parent proxies and origin servers based on URIs; certain types of URIs may cause requests to be sent through a special proxy server for specialized protocol handling.

## How Proxies Get Traffic

Since web browsers (ex: safari, google chrome, arc) typically communicate directly with web servers, you must first establish how HTTP traffic goes through the proxy. No matter how powerful a proxy is, it's useless if traffic just passes through it without going through the proxy. In order for a proxy to do its job, it must force web traffic from the client to go through the proxy. There are two main ways to direct this traffic: server-side control and client/network control.

In this article, we're going to take a closer look at the server-side control method in particular.\*\*

### DNS modifications

This is the most common way for reverse proxies (or sirgates) to receive traffic. The **core principle** is that the reverse proxy takes over the identity of the real web server.

Here's how it works

1. DNS record setup: When an administrator sets up DNS records for a website's hostname (e.g. jinmu.me), they register the reverse proxy's IP address instead of the real web server's IP address.
2. Client's DNS lookup: When a user types jinmu.me in a browser, the client's computer asks the DNS server for the IP address of this domain.
3. return the proxy IP address: The DNS server returns the IP address of the reverse proxy as set in the response.
4. Send request to proxy: **The client sends an HTTP request to that address, completely unaware that this IP address belongs to the proxy**. \*\*To the client, the proxy is the jinmu.me web server itself.

If you're interested in learning more about the behavior of DNS, I highly recommend checking out my [DNS Series](/series)!

Thanks to this approach, the client doesn't need to know anything and the server architecture is free to change.

### HTTP Redirection

Another way is for the web server to directly "tell" the client to use a proxy.

1. the client requests a resource from the web server.
2. Instead of processing the request directly, the web server sends a response with a special HTTP status code of **305 Use Proxy**.
3. The Location header in this response contains the address of the proxy server that the client should use.
4. The client that receives this response should resend the same request through the proxy server specified in the Location header.

However, this `305` status code is **now rarely used, and has been deprecated by many modern browsers for security reasons** because of the potential risk that a malicious server could be directed to a proxy that could spy on the user's traffic. So while it exists in concept, it is rarely seen in the real world.

### Client and network-side control methods

This is the opposite, controlling traffic flow directly from the client, the origin of the traffic, or from the network, the intermediate path. It is mainly used by **forward proxies**.

- Direct Route Setup (Client Settings): This is the most classic method. You can either go into your web browser settings and manually tell it to "connect to the internet via this proxy server", or you can use a script (PAC file) with specific rules to dynamically decide whether to use a proxy based on the URL.

- Network modification (transparent proxy): Unbeknownst to the client or user, **network equipment (routers, switches) in the way cast an invisible net** and force HTTP traffic to the proxy server. It's also called a "transparent proxy" because it's invisible to the user.

## Disadvantages

As powerful as proxies are, as with any technology, where there's light, there's shadow, and when we say "he controls everything," we also mean "everything depends on him." There are a few downsides to proxies that must be considered when introducing them into your architecture.

### Single Point of Failure (SPOF)

This is the most fatal weakness of proxies. All traffic must go through the proxy, which means that if the proxy server goes down, the entire service can be paralyzed. This is why it's practically essential in production to have redundancy by having multiple proxy servers, or to use a service that guarantees its own high availability, such as a load balancer in a cloud environment.

### Performance bottlenecks and complexity

Proxies add a very small amount of latency as they process and forward requests in the middle. In most cases, this is negligible, but if the proxy server is under-specified or its settings are not optimized, it can become a bottleneck that slows down the performance of the entire system. Additionally, proxy hierarchies or complex routing rules can add to the complexity of the architecture, making it very difficult to set up and manage.

### Security and privacy concerns

Proxies introduced to enhance security can paradoxically become a security hole. With all of the traffic being concentrated, if the proxy server itself is hacked, the entire system is at risk. Especially if SSL/TLS traffic is decrypted (SSL Termination) to inspect the contents, the proxy has access to all of the unencrypted sensitive data. This puts a lot of responsibility on the administrator, and creates another challenge to ensure the security of the proxy server itself is tightly managed.

## Outro

So, we started with the definition of a proxy, then moved on to why they are used, where they are found in various locations, how they control traffic, and finally the downsides.
This post started with the regret of not being able to answer the question about reverse proxy properly in my AUSG interview, but I was able to go beyond the superficial understanding of "a reverse proxy is something you put in front of a web server" and understand why it needs to be there and how it works so that the client is communicating with the proxy without realizing it. I'm reminded that the process of researching and refining my writing is the surest way to learn.

I hope this article will be helpful to those who, like me, were vaguely aware of proxies in the past. Thank you.

### Reference

- [Http definitive guide](https://dl.ebooksworld.ir/books/HTTP.The.Definitive.Guide.Brian.Totty.David.Gourley.OReilly.9781565925090.EBooksWorld.ir.pdf)

```embed
title: "[10-minute Tektalk] 🐿 Jamie's Forward Proxy, Reverse Proxy, Load Balancer"
image: "https://i.ytimg.com/vi/YxwYhenZ3BE/maxresdefault.jpg"
description: "🙋‍♀️ This is a 10-minute tech talk from the crew at ElegantTechCourse. 🙋‍♂️'10 Minute Tech Talks' is a time for the Elegant Tech Course crew to share and talk about what they've learned with their peers during the course. It's a time to share knowledge, talk, and reflect to help each other grow, and it's a self-directed..."
url: "https://www.youtube.com/watch?v=YxwYhenZ3BE"
```

```embed
title: "Proxies"
image: "https://i.ytimg.com/vi/Je3PMpvwjtE/maxresdefault.jpg"
description: "#Developers #Proxy #몰컴⭐️ CodeFactory integration links (all courses and books) ⭐️https://links.codefactory.ai👉 NestJS Masterclasshttps://bit.ly/fcnestjs👉 Flutter Beginnerhttps://inf.run/Sjuw👉 Flutter Intermediatehttps://..."
url: "https://www.youtube.com/watch?v=Je3PMpvwjtE"
```
