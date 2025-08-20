---
title: 매직 패킷은 왜 MAC주소를 16번 반복할까?
date: 2025-08-17T12:25:50.063Z
draft: false
tags:
  - WOL
  - WakeOnLan
  - MagicPacket
  - Network
  - Ethernet
  - Hardware
category: network
lastmod: 2025-08-20T09:40:21+09:00
layout: 
banner: 
---
최근 사용하지 않는 컴퓨터를 이용해서 proxmox 서버를 구축했다. 재밌게 홈서버를 구축하던 도중 문득, "원격으로 컴퓨터를 끄는 건 ssh 연결해서 `shutdown`이나 `halt`, `poweroff`같은 걸 사용할 수 있는데 꺼져 있는 컴퓨터를 원격으로 켜는 것도 가능할까?" 라는 생각이 들었다. 

그래서 알아봤더니  `WoL(Wake-on-LAN)`이라는 기술을 통해 구현할 수 있다고 해서 알아보던 도중 굉장히 궁금하게 생긴 부분을 발견해서 블로그에 간단히 적어보려 한다. 

WoL은 네트워크를 통해 저전력 상태의 컴퓨터에 신호를 보내 깨우는 기술이다. 그리고 이를 가능하게 하는 것이 **매직 패킷(Magic Packet)** 이라는 데이터 조각인데, 매직 패킷의 구조를 들여다 보면 컴퓨터의 고유 주소인 **MAC주소를 무려 16번이나 똑같이 반복**해서 보낸다. 4번 8번도 아니고 16번을 보내는 이런 신기한 숫자가 어떻게 나왔는지 궁금해서 그 이유를 파헤쳐 보려 한다.

## WoL과 매직 패킷

먼저 **WoL(Wake on LAN)** 과  **매직 패킷(Magic Packet)** 이 무엇인지 간단히 짚고 넘어가보자. 

- **WoL(Wake on LAN)**: WoL은 매직 패킷 기술에 부여된 프로토콜 이름이다.  이 기술은 원격으로 전원이 꺼져 있거나 절전 모드에 있는 PC를 네트워크를 통해 깨우는데 사용된다. 

- **매직 패킷(Magic Packet)**: 특수한 데이터 패턴을 가진 이더넷 프레임. 이 패킷은 수신 이더넷 컨트롤러에 의해 감지되어 컴퓨터를 꺠우는 역할을 한다. 

## 매직 패킷(이더넷 프레임)의 구조 

![[wake-on-lan-mac-address-16-times-1755498497920.webp]]

앞서 말했듯이 매직 패킷은 특수한 데이터 패턴을 가진 **이더넷 프레임**이다. 매직 패킷의 구조를 이해하기 위해서는 이더넷 프레임에 대해 알아야 한다. 

이더넷 프레임은 총 5가지 필드로 정의 된다. 

1. MAC dest : 목적지 MAC 주소. 이 패킷을 수신할 네트워크 장비의 물리적 주소를 의미한다. 보통 네트워크 내의 모든 장치에 신호를 보내기 위해 브로드캐스트 주소 `FF:FF:FF:FF:FF:FF`가 사용된다.
2. MAC src: 출발지 MAC 주소. 이 패킷을 보내는 장치의 MAC 주소이다. 
3. MISC(기타등등): 이더넷 프레임의 유형을 나타내는 이더타입(EtherType)등이 포함되는 영역이다. 이 패킷이 어떤 종류의 데이터인지를 알려주는 역할을 한다. 
4. Data / Payload: 실제 전송하려는 데이터가 담기는 부분이다. 상위 계층(ex: TCP/IP)의 헤더와 데이터가 여기에 포함된다. 만약 데이터의 크기가 46바이트보다 작을 경우, 패딩을 추가하여 최소 크기를 맞춘다. 
5. CRC(Cyclic Redundancy Check): 프레임 검사 시퀀스로 전송 중에 발생할 수 있는 오류를 검출하기 위한 필드이다. 해시 알고리즘 비슷하게 무결성을 확인하는 것 같은데 자세한 건 나중에 조금 더 알아보자.

> [!note]+ 이더넷 프레임의 패딩
> 초창기 이더넷은 모든 컴퓨터가 하나의 통신 회선을 공유하는 방식이었고 이때 **CSMA/CD**라는 규칙에 따라 통신했다. 이때 CD가 Collision Detection인데 만약 두 컴퓨터가 동시에 데이터를 보내서 '충돌'이 발생하면, 이 충돌을 발신 측에서 감지하고 데이터 전송을 멈춘 뒤 다시 보냈다.
> 이더넷 프레임의 데이터가 46바이트보다 작을 때 46바이트를 채워야 하는 이유는 충돌 감지(Collision Dectection)때문이다. 데이터 패킷이 너무 작으면 네트워크에서 데이터 충돌이 발생해도 발신 측이 이를 알아채지 못하고 다시 보내지 않고 지나가는 문제가 생길 수 있었다. 
> 충돌이 발생한 데이터는 다시 발신 측에 되돌아오게 되는데, 만약 A가 너무 작은 패킷을 보내 순식간에 전송을 끝내버리면 A는 데이터가 깨졌다는 사실을 모르고 데이터는 유실되고 만다. 
> 따라서 엔지니어들은 이런 시나리오에서도 충돌을 확실히 감지할 수 있도록, 패딩을 추가하여 임의로 데이터의 길이를 늘이는 방법을 택했다. 이때 정의된 크기가 바로 64바이트였고 프레임의 기본 정보 18바이트를 제외하고 실제 데이터의 최소 크기 (64 - 18)를 46바이트로 산정한 것이다. 

### Magic Pattern

그림에 나와 있는 **MAGIC Pattern**은 Payload 필드에 담기게 된다. 매직 패턴은 총 102바이트로 구성되며 크게 두 부분으로 나눌 수 있다. 

- FF ... FF (6 bytes): 동기화 스트림(Synchronization Stream)이라고 부르며 `FF`값이 6번 연속으로 나타나는데, 이 스트림은 수신측의 이더넷 컨트롤러가 매직 패킷을 쉽게 스캔하고 감지하도록 돕는다. 
- MAC 6 \* 16 (96bytes): **깨우고자 하는 MAC주소를 16번 반복**한 부분이다. 이부분이 매직 패턴의 핵심이다. 

추가로 Secure On Password로 6bytes크기의 비밀번호를 설정할 수 있는데, 일부 WoL기술에서 보안을 강화하기 위해 사용하는 영역이다. 기능이 활성화된 경우에만 사용되며, 허가되지 않은 사용자가 컴퓨터를 원격으로 켜는 것을 방지한다. 

> [!note]+ 동기화 스트림이 어떻게 쉽게 감지를 돕는가.
> 동기화 스트림은 스캔 상태 머신(scanning state machine)을 훨씬 단순하게 만들어준다. 스캔 상태 머신은 이더넷 컨트롤러 내부의 하드웨어 로직을 의미하는데, 이는 들어오는 데이터를 분석하여 특정 패턴을 찾는 역할을 한다. 
> 만약 동기화 스트림이 없다면, 이더넷 컨트롤러는 들어오는 모든 바이트 시퀀스에서 언제 6바이트 MAC 주소가 16번 반복되는 패턴이 시작될지 계속해서 확인해야 한다. 이는 보다 더 복잡하고 전력 소모가 큰 작업이 된다. 
> 하지만 FF ... FF라는 명확하고 독특한 동기화 스트림이 먼저 오면, 컨트롤러는 이 6바이트 시퀀스가 감지되었을 때 비로소 뒤이어 오는 MAC 주소 패턴을 스캔하기 시작할 수 있다. 즉, 동기화 스트림은 일종의 신호탄 혹은 표식 역할을 하여, 컨트롤러가 불필요한 스캔 작업을 최소화하고 필요한 시점에만 집중적으로 패턴을 찾도록 돕는다. 

사족이 길었지만, 이제 왜 MAC주소를 16번이나 길게 늘여 보내는지 알아 보자. 

## 왜 MAC주소를 16번이나 반복할까?


이에 대한 해답을 찾아 국내 글들을 찾아다녔지만, 이런 패킷을 보내면 이렇게 됩니다! 와 같이 나올 뿐 해답이 없었고 Gemini한테 물어봐도 아래와 이상한 답변한 할 뿐이었다. 

참고로 질문은 **매직 패킷에서 왜 mac 주소를 다른 숫자도 아니고 16번 반복하는 거야?** 이렇게 질문했다.

![[remote-wakeonlan-1755481071615.webp]]
~~박수짝짝~~ 

그러던 중 serverfault라는 사이트의 2014년 글을 보고 어느정도 이 물음에 대한 해답이 되었다. 사실 공식 문서도 아니고 정확한 정보도 아니지만 그래도 답변자의 논리에 내가 납득할 수 있었기 때문에 글을 가져와 봤다.

```embed
title: "Why does a Wake-On-LAN packet contain 16 duplications of the target MAC address?"
image: "https://cdn.sstatic.net/Sites/serverfault/Img/apple-touch-icon@2.png?v=9b1f48ae296b"
description: "From the Wireshark webpage: The Target MAC block contains 16 duplications of the IEEE address of the target, with no breaks or interruptions. Is there any specific reason for the 16 duplications?"
url: "https://serverfault.com/questions/632908/why-does-a-wake-on-lan-packet-contain-16-duplications-of-the-target-mac-address"
```

핵심은 구현의 **단순성**과 **신뢰성**이다. 

처음 매직 패킷을 만들던 AMD와 HP(Hewltt Packard)는 이 기능을 기존 하드웨어의 구성을 최대한 건들지 않고 추가하는 것을 목표로 했다. 따라서 이미 이더넷 컨트롤러에 존재하는 **주소 일치 회로**를 매직 패킷 감지에도 활용하기로 하였다. 
주소 일치 회로는 네트워크상에서 들어오는 수많은 이더넷 프레임 중에서 **해당 PC의 MAC 주소와 일치하는 프레임만을 식별하여 수신**하는 역할을 한다. 매직 패킷을 감지하는 기능을 추가하기 위해서는 단순히 MAC 주소를 16번 반복하여 세는 카운터를 추가하면 되었다. 

그래서 0부터 15까지의 값을 담을 수 있는 4비트 카운터를 이용하여, 카운터가 오버플로우(15+1=0)될 때 깨우기 코드를 실행하도록 하는 단순한 로직을 가능하게 한다. 위 링크의 pseudo code에 따르면 다음과 같다.

```text
Initialize a single counter that holds values from 0-15.
Set it to 0.
Watch the network. When I see the signal:
Loop:
  Do I see my address at the right spot?
  Yes: Add 1 to counter.
    Did I just overflow? (15+1 = 0?)
    Yes: Jump out of loop to "wake up" code.
...otherwise
Loop again.
```

여기까지가 구현의 단순성에 관한 것이고 **16이라는 숫자는 구현의 신뢰성에서 기인**한다. 앞서 말했듯이 인터넷은 수많은 정보가 왔다 갔다 하는 상황이다. 이런 상황에서 무작위 데이터 스트림이 시스템을 켜는 명령이 되는 것을 방지하기 위해서는 **쉽게 식별할 수 있어야 하며 오탐 가능성이 거의 없어야** 한다.

### 만일 2비트 카운터를 사용한다면?

만일 2비트 카운터를 사용하여 4번만 MAC 주소를 반복한다면 24 bytes이므로 프레임 데이터의 최소 크기(46bytes) 보다 작아 패딩을 추가하게 된다. 이렇게 된다면 우연히 해당 시퀀스가 네트워크에서 발생할 확률이 훨씬 높아져 **시스템이 의도치 않게 깨어날 가능성이 크게 증가했을 것**이다. 또한 **매직 패킷이 다른 네트워크 트래픽과 구별되는 특성이 약화**되어 신뢰성이 저하될 수 있다.
요약하자면, 2비트 카운터를 사용하여 MAC 주소를 4번만 반복하게 된다면 하드웨어 구현의 단순성 자체는 유지될 수 있었겠지만, 중요한 목표 중 하나인 **"오작동 방지"** 와 **"높은 신뢰성"** 측면에서 현재의 16번 반복 방식보다 현저히 불리했을 것이다.

### 만일 8비트 카운터를 사용한다면?

8비트 카운터는 0부터 255까지의 값을 표현할 수 있고 카운터가 오버플로우(255+1=0)되도록 하려면 MAC 주소는 256번 반복되어야 한다. MAC 주소는 6바이트이므로, 256번 반복되면 **1,536바이트**가 된다. 이는 이더넷 프레임의 최대 크기를 초과한 값이므로 실제 구현이 불가능하며 만일 가능하다 하더라도 프레임의 크기가 불필요하게 커져 무의미하게 많은 데이터를 옮겨야 했을 것이다. 

### 4비트 카운터 

그렇다면 왜 4비트 카운터였을까? 앞서 살펴본 가설들을 종합해보자

- **2비트 카운터(4번 반복)는 너무 위험하다.** 페이로드 크기가 너무 작아 오탐(false positive)의 가능성을 높인다. 네트워크상의 데이터가 우연히 매직 패킷으로 인식되어 시스템이 원치 않게 켜질 위험이 있었다. **신뢰성** 측면에서 탈락.
- **8비트 카운터(256번 반복)는 너무 비현실적이다.** 페이로드 크기가 이더넷 최대 전송 단위(MTU)를 초과해버려 애초에 표준 네트워크 환경에서 전송 자체가 불가능하다. **실용성** 측면에서 탈락.

결국 **4비트 카운터(16번 반복)** 는 이 두 가지 문제점을 모두 해결하는 최적의 지점이었다. 96바이트라는 충분한 길이를 확보하여 **신뢰성**을 극대화하면서도, 이더넷 프레임 표준을 완벽히 준수하여 **실용성**을 잃지 않았다. 동시에 4비트라는 단순한 카운터로 구현할 수 있어 **하드웨어의 복잡성**도 최소화했다.

이처럼 주어진 제약 조건 안에서 안정성과 효율성, 그리고 단순성까지 모두 잡아낸 숫자가 16이었던 것이다.
## Conclusion

단순히 홈서버를 구성하며 시작된 16이라는 숫자에 대한 호기심에서 비롯되어 여기까지 와버렸다. 초창기 이더넷 컨트롤러의 제한된 자원, 최소 프레임 크기, 그리고 무작위 데이터와 신호를 구분해야 하는 여러 제약 속에서 엔지니어들은 해법을 찾아내었다. 
사소해 보이는 기술적 디테일 하나에도 이처럼 합리적인 이유와  해법이 담겨 있다는 사실이, 엔지니어링의 진정한 매력이 아닐까 하는 생각이 든다. 




## Reference

```embed
title: "WakeOnLAN - Wireshark Wiki"
image: "https://www.wireshark.org/assets/img/wireshark-logo.png"
description: "WakeOnLAN is the protocol name given to the so-called Magic Packet technology, developed by AMD and Hewlett Packard for remotely waking up a remote host that may have been automatically powered-down because of its power management features. Although power management allows companies and individuals to cut power usage costs, it presents a problem for IT departments especially in being able to quickly and efficiently remotely manage PC’s, especially during off-hours operation when those PC’s are most likely to be in a suspended or standby state, assuming power management features are enabled."
url: "https://wiki.wireshark.org/WakeOnLAN"
```

```embed
title: "Wake-on-LAN - Wikipedia"
image: "https://upload.wikimedia.org/wikipedia/commons/thumb/5/58/Wake_on_LAN_connector.JPG/1200px-Wake_on_LAN_connector.JPG"
description: "Wake-on-LAN (WoL)[a] is an Ethernet or Token Ring computer networking standard that allows a computer to be turned on or awakened from sleep mode by a network message. The message is usually sent to the target computer by a program executed on a device connected to the same local area network (LAN). It is also possible to initiate the message from another network by using subnet directed broadcasts or a WoL gateway service. It is based upon AMD’s Magic Packet Technology, which was co-developed by AMD and Hewlett-Packard, following its proposal as a standard in 1995. The standard saw quick adoption thereafter through IBM, Intel and others."
url: "https://en.wikipedia.org/wiki/Wake-on-LAN"
```

```embed
title: "20213.pdf"
image: "https://t3.gstatic.com/faviconV2?client=SOCIAL&type=FAVICON&fallback_opts=TYPE,SIZE,URL&url=https://www.amd.com/content/dam/amd/en/documents/archived-tech-docs/white-papers/20213.pdf&size=128"
description: "Publication# 20213"
url: "https://www.amd.com/content/dam/amd/en/documents/archived-tech-docs/white-papers/20213.pdf"
```
