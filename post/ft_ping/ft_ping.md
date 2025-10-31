---
title: ping을 재구현 해보자.
date: 2025-08-20T01:01:34.662Z
tags: 
category: 
draft: false
lastmod: 2025-08-20T13:14:01+09:00
layout: 
series: 
seriesOrder: 
banner: 
---

## RAW Socket

RAW 소켓은 운영체제의 전송 계층(TCP/UDP 계층)을 거치지 않고, 네트워크 계층(IP 계층)의 데이터를 직접 읽고 쓸 수 있게 해주는 소켓이다.

> [!note] IP의 특징
>

## ICMP

Internet Control Message Protocol

### Message Type

ping은 그 중에서도

type 8 Echo Request
type 0 Echo Reply

를 사용

## Ping

Ping은 빠르고 간단한 확인이 목적.
만약 TCP를 사용한다면 3-way handshake를 거쳐야 한다.
ICMP는 이런 과정 없이 단 두 개의 메시지만으로 목적을 달성할 수 있어 매우 가볍고 효율적이다.

ICMP는 네트워크 계층(IP)에서 직접 동작한다. 이것이 매우 중요한데, 목적 서버의 웹 서버나 데이터베이스 같은 특정 애플리케이션이 응답하지 않더라도, 서버의 운영체제만 정상적으로 작동하고 있다면 ICMP Echo 요청에는 응답할 수 있다.

따라서 `ping`은 특정 서비스가 아닌, 네트워크에 연결된 호스트 자체의 기본적인 생존 여부를 확인하는 가장 근본적인 테스트가 된다.
