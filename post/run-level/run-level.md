---
title: Run Level은 뭐고 왜 필요할까?
date: 2025-08-11T12:35:55.661Z
tags:
  - Linux
  - RunLevel
  - SystemAdmin
  - SysVinit
  - Systemd
category: develop
draft: true
lastmod: 2025-08-19T15:06:34+09:00
layout: 
summary: Linux 시스템에서 런레벨(Run Level)의 개념과 각 레벨의 특징, 그리고 현대적인 systemd 타겟(Target) 시스템으로의 전환까지.
series: 
seriesOrder: 
banner: 
---

## Intro

리눅스 마스터 자격증을 준비하던 중 **Run Level**이라는 다소 생소한 단어를 보게됐다.  
평소 Ubuntu나 CentOS를 사용하면서 시스템이 때로는 GUI로, 때로는 터미널로 부팅되는 걸 당연하게 여겼는데,  이것들이 런레벨이라는 규칙에 의해서 동작하는 것을 알게되어서 정리하려한다.

## Run Level이란?

**Run Level(런레벨)** 은 Unix/Linux 기반 운영체제에서 `init` 프로세스의 운영 상태를 의미 한다. 시스템이 부팅될  때 어떤 서비스와 프로세스로 시작할지 정의하는 역할을 한다. 쉽게 말해, 처음 컴퓨터를 켤 때 '어떤 목적'으로 부팅할지 미리 정해놓은 여러 가지 시나리오라고 생각할 수 있다.

런레벨은 이런 시나리오들에 번호를 매겨 **시스템이 어떤 모드로 동작하는지를 숫자로 나타낸 것**이다. 각 런레벨마다 다른 서비스 조합이 실행되며, 시스템 관리자는 이를 통해 시스템의 동작을 제어할 수 있다.

Linux 시스템에서는 전통적으로 0부터 6까지 7개의 런레벨이 정의되어 있다. 각 런레벨의 특징을 살펴보자:

### Run Level 0 - 시스템 종료

- **목적**: 시스템을 안전하게 종료
- **특징**: 모든 프로세스를 정리하고 시스템을 완전히 종료
- **사용 예시**: `init 0` 또는 `shutdown -h now`

### Run Level 1 - 단일 사용자 모드

- **목적**: 시스템 관리 및 유지보수
- **특징**:
  - root 사용자만 로그인 가능
  - 네트워크 서비스 비활성화
  - 최소한의 시스템 서비스만 실행
- **사용 예시**:
  - 잊어버린 root 비밀번호 복구
  - 시스템 파일 복구
  - 중요한 설정 파일 수정

window의 안전모드와 비슷한 개념이다.

### Run Level 2 - 다중 사용자 모드 (NFS 없음)

- **목적**: 로컬 다중 사용자 환경
- **특징**:
  - 여러 사용자 로그인 가능
  - 네트워크 파일 시스템(NFS) 서비스는 비활성화
  - 로컬 네트워킹만 지원

> [!note] NFS (Network File System)
> 네트워크를 통해 파일을 공유하는 시스템이다.

### Run Level 3 - 완전한 다중 사용자 모드

- **목적**: 서버 환경에서 주로 사용
- **특징**:
  - 모든 네트워크 서비스 활성화
  - 명령줄 인터페이스(CLI)만 제공
  - 그래픽 인터페이스 없음
- **사용 예시**: 웹 서버, 데이터베이스 서버 등

### Run Level 4 - 사용자 정의

- **목적**: 사용자가 임의로 정의
- **특징**: 일반적으로 사용되지 않으며, 특별한 목적으로 커스터마이징 가능

### Run Level 5 - 그래픽 다중 사용자 모드

- **목적**: 데스크톱 환경
- **특징**:
  - 모든 네트워크 서비스 활성화
  - 그래픽 사용자 인터페이스(GUI) 제공
  - X Window 와 같은 X기반의 로그인.

### Run Level 6 - 시스템 재부팅

- **목적**: 시스템 재시작
- **특징**: 모든 프로세스를 정리하고 시스템을 재부팅
- **사용 예시**: `init 6` 또는 `reboot`

## Systemd와 Target 시스템

현대적인 Linux 배포판(RHEL 7+, Ubuntu 16.04+, CentOS 7+ 등)에서는 전통적인 SysVinit 대신 **systemd**를 사용합니다. systemd에서는 런레벨 대신 **Target** 개념을 사용합니다.

### 주요 Systemd Target들

| 전통적인 Run Level | Systemd Target    | 설명            | 추가 특징                         |
| -------------- | ----------------- | ------------- | ----------------------------- |
| 0              | poweroff.target   | 시스템 종료        | 안전한 종료 절차 수행                  |
| 1              | rescue.target     | 단일 사용자 모드     | 복구 작업용, 최소 서비스만 실행            |
| 2,3,4          | multi-user.target | 다중 사용자 CLI 모드 | 네트워크 포함, GUI 없음               |
| 5              | graphical.target  | 그래픽 다중 사용자 모드 | multi-user.target + 디스플레이 매니저 |
| 6              | reboot.target     | 시스템 재부팅       | 안전한 재시작 절차 수행                 |

### Target의 계층 구조

systemd target은 계층적으로 구성되어 있어, 상위 target이 하위 target들을 포함합니다:

현대적인 Linux 배포판(RHEL 7+, Ubuntu 16.04+, CentOS 7+ 등)에서는 전통적인 SysVinit 대신 **systemd**를 사용합니다. 이는 단순한 도구 교체가 아닌 시스템 초기화 방식의 근본적인 변화를 의미합니다.

### 왜 변화가 필요했을까?

#### SysVinit의 한계점

전통적인 SysVinit 시스템은 1980년대부터 사용되어 온 안정적인 시스템이지만, 현대적인 요구사항에 몇 가지 한계를 드러냈습니다:

1. **순차적 부팅의 비효율성**

   ```bash
   # SysVinit: 서비스들이 순차적으로 시작
   S01network -> S02firewall -> S03database -> S04webserver
   # 각 서비스가 완전히 시작될 때까지 기다린 후 다음 서비스 시작
   ```

2. **의존성 관리의 복잡성**
   - 서비스 간 의존성을 스크립트 번호로만 관리
   - 동적 의존성 해결 불가
   - 복잡한 의존성 트리에서 오류 발생 시 디버깅 어려움

3. **하드웨어 변화에 대한 대응 부족**
   - USB, 핫플러그 장치 등 동적 하드웨어 지원 미흡
   - 네트워크 인터페이스 변화에 대한 유연한 대응 어려움

4. **로그 관리의 분산**
   - 각 서비스마다 다른 로그 형식과 위치
   - 시스템 전체 상태 파악의 어려움

#### Systemd의 혁신적 접근

systemd는 이러한 문제들을 다음과 같이 해결했습니다:

1. **병렬 부팅**

   ```bash
   # systemd: 의존성이 없는 서비스들을 동시에 시작
   network.service + firewall.service + basic.target
   └─ database.service + webserver.service (network 완료 후)
   ```

2. **정교한 의존성 관리**

   ```ini
   # systemd unit 파일 예시
   [Unit]
   Description=Web Server
   After=network.target database.service
   Wants=network.target
   Requires=database.service
   ```

3. **이벤트 기반 시스템**
   - D-Bus를 통한 실시간 상태 모니터링
   - 하드웨어 변화에 즉시 반응
   - 소켓 기반 서비스 활성화

4. **통합 로깅 시스템**

   ```bash
   # 모든 시스템 로그를 하나의 인터페이스로
   $ journalctl -u nginx.service
   $ journalctl --since "1 hour ago"
   ```

### Run Level vs Target: 개념적 차이

| 측면 | SysVinit Run Level | Systemd Target |
|------|-------------------|-----------------|
| **구조** | 선형적, 단일 상태 | 병렬적, 다중 상태 가능 |
| **의존성** | 숫자 순서 기반 | 명시적 의존성 선언 |
| **확장성** | 7개 고정 레벨 | 무제한 target 생성 가능 |
| **활성화** | 한 번에 하나의 런레벨 | 여러 target 동시 활성화 |

```bash
# SysVinit: 하나의 런레벨만 활성
$ runlevel
N 3

# systemd: 여러 target이 동시에 활성
$ systemctl list-units --type=target --state=active
basic.target      loaded active active Basic System
multi-user.target loaded active active Multi-User System
network.target    loaded active active Network
```

### 실제 성능 차이

부팅 시간 비교 (일반적인 서버 환경):

- **SysVinit**: 약 60-120초
- **systemd**: 약 15-30초

이러한 성능 향상은 병렬 처리와 지연 로딩, 소켓 활성화 등의 기술 덕분입니다.

systemd에서는 런레벨 대신 **Target** 개념을 사용합니다. Target은 단순히 런레벨의 대체재가 아니라, 더 유연하고 강력한 시스템 상태 관리 방식입니다.

### 주요 Systemd Target들

| 전통적인 Run Level | Systemd Target | 설명 | 추가 특징 |
|-------------------|----------------|------|-----------|
| 0 | poweroff.target | 시스템 종료 | 안전한 종료 절차 수행 |
| 1 | rescue.target | 단일 사용자 모드 | 복구 작업용, 최소 서비스만 실행 |
| 2,3,4 | multi-user.target | 다중 사용자 CLI 모드 | 네트워크 포함, GUI 없음 |
| 5 | graphical.target | 그래픽 다중 사용자 모드 | multi-user.target + 디스플레이 매니저 |
| 6 | reboot.target | 시스템 재부팅 | 안전한 재시작 절차 수행 |

### Target의 계층 구조

systemd target은 계층적으로 구성되어 있어, 상위 target이 하위 target들을 포함합니다:

```bash
graphical.target
├── multi-user.target
│   ├── basic.target
│   │   ├── sysinit.target
│   │   └── sockets.target
│   ├── network.target
│   └── remote-fs.target
└── display-manager.service
```

이러한 구조 덕분에:

- `graphical.target`을 활성화하면 자동으로 `multi-user.target`도 활성화
- 필요한 의존성이 자동으로 해결됨
- 부분적인 상태 전환이 가능함

### Systemd에서 Target 관리하기

```bash
# 현재 기본 target 확인
$ systemctl get-default

# 기본 target 변경
$ sudo systemctl set-default multi-user.target

# target 즉시 변경
$ sudo systemctl isolate graphical.target

# target의 의존성 확인
$ systemctl list-dependencies graphical.target

# 특정 target으로 부팅 (GRUB에서)
# 커널 매개변수에 systemd.unit=rescue.target 추가
```

## 실전 활용 사례

### 1. 그래픽 인터페이스 문제 해결

GUI에 문제가 생겨 시스템에 로그인할 수 없을 때:

```bash
# 부팅 시 GRUB에서 'e' 키를 눌러 편집 모드 진입
# linux 라인 끝에 추가:
systemd.unit=multi-user.target

# 또는 전통적인 방법:
3
```

### 2. 서버 최적화

서버 환경에서 불필요한 GUI 서비스 제거:

```bash
# GUI 제거하고 CLI만 사용
$ sudo systemctl set-default multi-user.target
$ sudo systemctl disable gdm  # Display Manager 비활성화
```

### 3. 시스템 복구

손상된 설정 파일 복구나 비밀번호 재설정:

```bash
# 단일 사용자 모드로 부팅
# GRUB에서 systemd.unit=rescue.target 추가
# 또는 traditional runlevel 1
```

## 보안 고려사항

런레벨을 사용할 때 고려해야 할 보안 사항들:

### 1. 최소 권한 원칙

- 필요한 서비스만 실행하는 런레벨 사용
- GUI가 필요 없는 서버에서는 런레벨 3 사용

### 2. 단일 사용자 모드 보안

- 물리적 접근 시 단일 사용자 모드로 부팅 가능
- GRUB 비밀번호 설정으로 보안 강화 필요

### 3. 서비스 관리

- 각 런레벨에서 실행되는 서비스 정기적 점검
- 불필요한 서비스 비활성화

## 트러블슈팅

### 잘못된 런레벨로 부팅된 경우

```bash
# 현재 런레벨 확인
$ runlevel

# 원하는 런레벨로 변경
$ sudo init 5  # GUI 모드로 변경
$ sudo init 3  # CLI 모드로 변경
```

### 부팅이 안 되는 경우

1. GRUB에서 복구 모드 선택
2. 단일 사용자 모드로 부팅
3. 설정 파일 확인 및 수정
4. 정상 런레벨로 재부팅

## 실무에서의 변화 영향

### 시스템 관리자가 알아야 할 변화점

1. **스크립트 위치 변경**

   ```bash
   # SysVinit 환경
   /etc/init.d/nginx start
   /etc/rc.d/rc3.d/S80nginx
   
   # systemd 환경
   systemctl start nginx
   /etc/systemd/system/nginx.service
   ```

2. **로그 확인 방법 변화**

   ```bash
   # 기존 방식
   $ tail -f /var/log/messages
   $ cat /var/log/nginx/error.log
   
   # systemd 방식
   $ journalctl -f
   $ journalctl -u nginx.service
   ```

3. **서비스 의존성 관리**
   - SysVinit: 수동으로 스크립트 번호 조정
   - systemd: unit 파일에서 의존성 명시

### 마이그레이션 전략

기존 시스템을 systemd로 전환할 때 고려사항:

1. **단계적 전환**
   - 중요한 서비스부터 systemd unit 파일 작성
   - 기존 init 스크립트와 병행 운영 기간 설정
   - 충분한 테스트 후 완전 전환

2. **문서화 및 교육**
   - 팀 내 systemd 교육 진행
   - 새로운 운영 절차 문서화
   - 트러블슈팅 가이드 업데이트

3. **모니터링 도구 조정**
   - 기존 모니터링 스크립트 수정
   - journald 로그 수집 체계 구축

## 마무리

Run Level과 systemd target은 Linux 시스템 관리의 핵심 개념입니다. SysVinit에서 systemd로의 변화는 단순한 도구 교체가 아닌 **시스템 철학의 진화**를 의미합니다.

### 핵심 메시지

1. **두 시스템 모두 이해하기**: 레거시 시스템과 현대적 시스템을 모두 다뤄야 하는 실무자라면 양쪽 모두 숙지 필요

2. **변화의 이유 이해하기**: systemd의 도입은 성능, 의존성 관리, 시스템 모니터링의 개선을 위한 필연적 선택

3. **실무 적용**: 문제 해결 시 단일 사용자 모드 활용법과 서비스 의존성 관리 방법을 숙지

특히 DevOps 환경에서는 컨테이너와 오케스트레이션 도구들이 systemd의 개념을 많이 차용하고 있어, 이러한 기본기가 더욱 중요해지고 있습니다. 시스템의 상태 관리와 서비스 의존성 해결은 현대적인 인프라 관리의 핵심 요소이기 때문입니다.
