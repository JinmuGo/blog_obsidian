---
title: Domain Name System (DNS) DeepDive - Name Resolution
date: 2025-07-06T02:48:10.982Z
draft: false
layout: PostDefault
summary:
banner: banner/dns-deepdive.jpg
tags:
  - DNS
lastmod: 2025-07-06T23:29:49+09:00
category: develop
series: DNS-DeepDive
seriesOrder: 3
---

Finding the IP address that corresponds to a domain name according to the user's needs is called name resolution. In this post, we will see the basic behavior of name resolution by DNS.

## Query and answer

The basis of Name Resolution is a query and a response. This query and response has three promises.

1. queries and responses always correspond 1:1.
   - You will never get a response that doesn't correspond to a query, or two responses to a query.
2. If you want to know information, you must specify the domain name and type of information you want to know.
   - The name and type must be specified together, such as "IP address of jinmu.me".
3. The person providing the information responds to the query with the information they know.

## Name Resolution follows a hierarchical structure

As we discussed in the previous post, DNS uses hierarchy and delegation to create a hierarchy with the root as the vertex. The basis of name resolution by DNS is to follow the order of the hierarchy, starting with the root name server at the top, and query it to get the information you want to know, and end up with the IP address you need. This behavior is called iterative resolution in DNS, and we'll refer to it as following the hierarchy.

![[image-142.webp]]

In order for a querier to follow the hierarchy of DNS and perform name resolution, they need to know the appropriate query source. This is why DNS has adopted a structure where a child registers its name server information with its parent, and the parent, upon receiving a query for a delegated domain name, responds with the child's name server information as the next query source. This structure allows the querier to follow the hierarchy by delegation.

![[image-1232.webp]]

### Preparation (left side of the figure).

1. A parent-child relationship is established between `root` and `me` by delegation. Root is the parent and me is the child.
2. me registers its name server information with root, the parent. Root registers and configures the name server information registered by me to its name servers.

### Real name resolution (right side of the figure)

1. while the parent-child relationship is established by delegation, A asks Root, "Please tell me the IP address of jinmu.me".
2. the root responds with the name server information of `me` (`b.dns.me`) because it only knows the name server information of the delegate for `me`.
3. A queries `b.dns.me` using the name server information for me that it received from root.

Following the DNS hierarchy in this way, starting at the root and working backward, is the basis of the name resolution structure.

## Name resolution in action

Let's take a look at the hierarchical name resolution we saw above in a little more detail.

![[dns-deepdive-3-1751809205331.webp]]

1.  query
    - The user sends a query to the name server at the top of the hierarchy, the root, specifying the name and kind of information they want to know.
2.  response
    - The root responds with the name server information of `me` (`b.dns.me` in this example) because it is delegating to `me`.
3.  query
    - From the response received from root, we can see two things
      - Root is delegating `me`.
      - The delegate's name server is `b.dns.me`.
    - Use this information to send a single query to the name server for `me`.
4.  Response
    - Because `me` is delegating to `jinmu.me`, it responds with the name server information of its delegate, `jinmu.me` (`ns1.jinmu.me` in this example).
5.  Query
    - From the response received from `me`, we can see two things
      - `me` is delegating `jinmu.me`.
      - The name server of the delegate is `ns1.jinmu.me`.
    - Using this information, send the same query as in step 1 to the name server at `jinmu.me`.
6.  Response
    - The name server of `jinmu.me` responds with the IP address because it knows the IP address of `jinmu.me`. This allows the user to get the IP address of `jinmu.me` (`76.76.21.21`) and the name resolution ends.

## Reducing the load and time of name resolution

While it is certainly possible to perform name resolution by following the DNS hierarchy from the root, it requires a lot of requests, responses, and time. Therefore, a solution was devised to have a separate server in charge of name resolution and accept requests from other hosts to perform name resolution on their behalf.

In our example, let's call this agent X. You send a request to X: "**Please resolve names for me** and give me the IP address of `jinmu.me\*\*", and X will do the name resolution described above.

![[dns-deepdive-3-1751810301365.webp]]

X responds to User with the IP address `76.76.21.21` of `jinmu.me` by performing the name resolution requested by A.

When X resolves a name, it stores the information from the querying name server for some time. If A makes the same query to X within that time, X already has the result, so it doesn't resolve a new name and responds to the user with the result it already has.

![[dns-deepdive-3-1751810710080.webp]]

This reduces the number of queries to the name servers, which reduces the load and time it takes to resolve names.

From the information so far, we can see that there are three different roles when it comes to DNS name resolution.

1. the person who needs the information (User A, User B, User C)
2. a person (X) who is doing the name resolution on behalf of the person who needs the information.
3. those who provide the information (the name servers that create the hierarchy of DNS)

## Outro

In this article, we looked at "name resolution," the process of finding the IP address corresponding to a domain name. We learned about the basic principle of "iterative resolution," which starts from the root name server and follows the hierarchy in order, and the emergence of name resolvers with cache to reduce the query and response load and time of this process. We saw that the DNS ecosystem consists of three roles: "users" who request information, "cache resolvers" who execute name resolution on behalf of users, and "authoritative name servers" who provide information.

So far, we've painted the big picture of "how" DNS works. In our next post, we'll take it a step further and dive deeper into the specific components of DNS that actually make this process possible, and what information they pass back and forth to each other.

Acknowledgments.
