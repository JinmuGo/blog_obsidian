---
title: Let's re-implement ping.
date: 2025-08-20T01:01:34.662Z
tags:
Category:
draft: false
lastmod: 2025-08-20T13:14:01+09:00
layout:
Series:
seriesOrder:
banner:
---

## RAW Socket

A RAW socket is a socket that allows you to read and write data from the network layer (IP layer) directly, bypassing the operating system's transport layer (TCP/UDP layer).

> [!note] Features of IP
>

## ICMP

Internet Control Message Protocol

### Message Type

ping uses the

type 8 Echo Request
type 0 Echo Reply

using

## Ping

Ping is intended for quick and simple verification.
If you use TCP, you have to go through a 3-way handshake.
ICMP is very lightweight and efficient, requiring only two messages to accomplish its purpose.

ICMP operates directly at the network layer (IP). This is very important because even if the target server's specific application, such as a web server or database, is unresponsive, it can still respond to ICMP Echo requests as long as the server's operating system is functioning properly.

This makes `ping' the most fundamental test of the basic survivability of the hosts connected to the network, rather than a specific service.
