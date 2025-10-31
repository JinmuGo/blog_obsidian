---
title: RAID
date: 2025-08-04T11:51:15.990Z
draft: true
layout: 
summary: RAID에 대해
tags:
  - RAID
lastmod: 2025-08-05T21:59:20+09:00
category: develop
---
Redundant Array of Independent(inexpensive) Disks

RAID란 **여러 개의 물리적 디스크를 하나의 논리적인 장치처럼 사용하여, 데이터의 안정성을 높이거나(신뢰성) 처리 속도를 향상시키는(성능) 디스크 배열 기술** 이다.

## RAID의 개요.
여러 개의 하드 디스크가 있을 때 동일한 데이터를 다른 위치에 중복해서 저장하는 방법.
데이터를 여러 개의 디스크에 저장하여 입출력 작업이 균형을 이루게 되어 전체적인 성능을 향상. 운영체제에서 하나의 RAID는 논리적으로 하나의 디스크로 인식하여 처리된다.

Parity

ECC(Error Check & Correction)
## RAID의 이용
초기의 RAID는 저용량 하드 디스크를 하나의 디스크로 확장하여 사용하는 것이 주류였으나 현재는 백업을 가능하게 하고 안정적인 데이터의 보존과 유지기능, 속도 향상 등에 사용한다.

단순히 여러 디스크를 하나로 묶어주는 기술로는 Linear가 있다.

최근에는 단순히 묶어주는 Linear 보다는 스트라이핑(Striping )이나 미러링(mirroring)기술이 적용된 RAID구성이 보편적이다.
RAID를 구성하는 방법도 소프트웨어, 하드웨어 구현 등 다양한 방법으로 가능하다.

## RAID에서 사용하는 기술

1. 스트라이핑
	-  
2. 미러링 




## 패리티 

XOR 연산 