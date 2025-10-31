---
title: Oracle에서 Bastion 서버 구성하기
date: 2025-08-16T01:40:54.498Z
tags: 
category: 
draft: true
lastmod: 2025-08-19T16:57:27+09:00
layout: 
series: 
seriesOrder: 
banner: 
---

지원되지 않는 이미지
Oracle Help Center에 문의하였지만 쉽지않았음.
최대한 지원되는 이미지로 구성하자.
그게 아니라면 직접 문의를 해서 Oracle cloud Agent를 설치하시길.

oracle cloud agent에서 bastion plugin 을 enable해야한다. 
enable하고 몇분이 지나면 status가 running이 된다.

현재 내 서버는 2개의 ampere서버로 나뉘어져 있다.
직접 연결을 하였지만 보안상의 이유로 


bastion이란?

```embed
title: "Bastion Overview"
image: "https://docs.oracle.com/en-us/iaas/Content/Bastion/images/bastion-overview-diagram.png"
description: "Oracle Cloud Infrastructure Bastion provides restricted and time-limited access to target resources that don’t have public endpoints."
url: "https://docs.oracle.com/en-us/iaas/Content/Bastion/Concepts/bastionoverview.htm"
```

