---
title: Zotero 동기화를 위한 WebDAV서버 배포하기 with Coolify
date: 2025-03-25T04:46:34.380Z
draft: false
layout: PostBannerX
summary: Coolify에 WebDAV 서버를 구축하고 여러 기기 간 문서 동기화를 구현한 경험.
tags:
  - zotero
  - coolify
  - webdav
lastmod:
bannerOnlyText: true
category: develop
---

## Intro

저는 서지 관리 프로그램으로 오래전부터 [Zotero](https://www.zotero.org)를 사용해 왔습니다. 하이라이팅, 메모, 캡처 기능과 더불어서 Obsidian과 플러그인을 통해 Integration까지 되어서 관리할 수 있어 유용하게 사용하고 있죠. 게다가 이번 Zotero7 업데이트로 인해 UI가 더 예쁘게 바뀌었더군요. 관심 있는 분들은 이 [유튜브 채널](https://www.youtube.com/@brain.trinity/videos)이 도움이 많이 되니 한번 알아보시는 걸 추천 드립니다.

![[2jgsz9.png]]
그렇게 Zotero를 사용하고 있던 와중. 문제가 하나 생겼습니다. 최근 영어 공부할 일이 생겨서 단어장을 pdf로 받아 평소처럼 Zotero에서 관리하고 있었는데, 이걸 갖고 있는 아이패드로도 이동하는 중에 보고 싶었습니다. 물론 다른 앱(good notes, files 등….)을 사용하면 되지만, 저는 무엇보다 하이라이팅과 메모, 기능을 사용하고 싶었고 한곳에서 관리하고 싶었습니다. 즉, 아이패드에서 했던 작업을 맥북, 휴대전화에서도 보고 싶었습니다. Zotero에서도 이런 기능을 제공합니다. [Zotero Storage](https://www.zotero.org/storage?id=storage)서비스를 이용하면 되는데, 이게 은근히 돈이 나갑니다. 공간 제한도 있고요.
![[eici5g.png]]

그래서 다른 대안을 찾기 위해 [Zotero sync](https://www.zotero.org/support/sync#webdav)문서를 보던 중 WebDAV를 이용한 File Sync 항목을 보게 되었습니다. WebDAV 프로토콜을 지원하는 서버만 있다면, zotero sync기능을 사용할 수 있다는 글이 적혀있었죠. 저는 이 방법이다! 라고 생각하며 구현하기로 하였죠. 하지만 그전에 먼저 WebDAV가 무엇인지 알아야 했습니다.

## WebDAV

WebDAV(Web Distributed Authoring and Versioning)는 **HTTP 프로토콜을 확장**하여 웹 서버에 **저장된 파일을 협업 편집 및 관리할 수 있게 해주는 표준 기술**입니다.

여기에서 주목할 만한 점은 아래와 같습니다.

1. [[#HTTP 프로토콜을 확장한 프로토콜]]이다.
2. [[#원격 파일 협업 편집 및 관리]] 할 수 있게 해준다.

### HTTP 프로토콜을 확장한 프로토콜

- **기존 HTTP 메소드 활용**: WebDAV는 HTTP의 GET, POST, PUT, DELETE 등 기존 메소드를 활용하면서, 파일 관리와 협업을 위한 새로운 메소드를 추가했습니다.
- **PROPFIND**: 파일 및 디렉토리의 메타데이터 조회
- **PROPPATCH**: 파일 속성 수정
- **MKCOL**: 디렉토리 생성
- **COPY**: 파일 복사
- **MOVE**: 파일 이동
- **LOCK/UNLOCK**: 파일 잠금 및 해제
- **VERSION-CONTROL**: 버전 관리
- **XML 기반 데이터 교환**: WebDAV는 XML을 사용하여 파일 메타데이터를 교환합니다. 이는 파일 속성, 잠금 상태, 버전 정보 등을 효율적으로 관리할 수 있게 합니다.

### 원격 파일 협업 편집 및 관리

- **파일 잠금(LOCK/UNLOCK)**: 다중 사용자 환경에서 파일 덮어쓰기를 방지하기 위해 파일을 잠글 수 있습니다. 이는 협업 시 중요한 기능으로, 다른 사용자가 파일을 편집 중일 때 충돌을 방지합니다.
- **속성 관리(PROPFIND/PROPPATCH)**: 파일의 메타데이터(생성 날짜, 수정 시간, 작성자 등)를 조회하고 수정할 수 있습니다. 이는 파일의 이력 관리와 검색 기능을 강화합니다.
- **네임스페이스 관리(COPY/MOVE)**: 서버 내 파일을 복사하거나 이동할 수 있습니다. 이는 파일 구조를 유지하면서 파일을 재배치할 때 유용합니다.
- **버전 관리**: WebDAV는 파일의 버전을 관리할 수 있는 기능을 제공합니다. 이는 문서의 변경 이력을 추적하고, 이전 버전으로 되돌릴 수 있게 합니다.

WebDAV에 대해 알아보니 왜 Zotero가 이 프로토콜을 통해 File Sync를 하는지 알 수 있었습니다. 문서 관리를 위한 메소드들이 적절하게 구현되어 있고 XML을 사용하여 필요한 정보만 주고받아 비교적 빠른 속도로 관리할 수 있었습니다. 이렇게 WebDAV에 대해 알아보았으니 이제 구현을 해봐야겠죠.

## WebDAV Server 구현

`Zotero webdav` 키워드로 구글 검색을 해보니 거의 외부 서비스를 이용하거나 시놀로지 NAS를 사용하더군요. [IaaS 외부 서비스](https://www.zotero.org/support/kb/webdav_services)는 보안이나 프라이버시 측면에서 꺼려지고 가지고 있는 NAS도 없어서 다른 방안을 찾아봐야 했습니다.
그래서, 기존에 구축해 둔 [OCI(Oracle Cloud Infrastructure)](https://www.oracle.com/kr/cloud/)에 인스턴스를 활용하여 WebDAV 서버를 호스팅하기로 했습니다. Zotero 문서에서 말하길 WebDAV 프로토콜을 지키는 서버만 있으면 File Sync가 된다고 하니 괜찮을 것 같았죠. 여러분들도 남는 컴퓨터가 있다면 따라 하실 수 있습니다. 그렇게 WebDAV 프로토콜을 지키는 간단한 서버를 구현한 Repository를 찾았습니다.

```embed
title: "GitHub - hacdias/webdav: A simple and standalone WebDAV server."
image: "https://opengraph.githubassets.com/834a81c8be8391c1ef62225a5b6bcb0d5e9f8db0c9fb95035d38836d5ae/hacdias/webdav"
description: "A simple and standalone WebDAV server. Contribute to hacdias/webdav development by creating an account on GitHub."
url: "https://github.com/hacdias/webdav"
```

conf하나로 간편하게 설정할 수 있고 무엇보다 docker image를 배포해 둔 상태여서 간편하게 다음 Dockerfile을 작성해서 사용할 수 있었습니다.

```dockerfile
FROM ghcr.io/hacdias/webdav:latest

ENTRYPOINT [ "webdav", "-c", "/webdav/config.yml" ]
```

배포한 서버의 Config 설정은 다음과 같습니다.

```yml
address: 0.0.0.0
port: 6065

# TLS-related settings if you want to enable TLS directly.
tls: false
cert: cert.pem
key: key.pem

# Prefix to apply to the WebDAV path-ing. Default is '/'.
prefix: /

# Enable or disable debug logging. Default is 'false'.
debug: false

# Disable sniffing the files to detect their content type. Default is 'false'.
noSniff: false

# Whether the server runs behind a trusted proxy or not. When this is true,
# the header X-Forwarded-For will be used for logging the remote addresses
# of logging attempts (if available).
behindProxy: false

# The directory that will be able to be accessed by the users when connecting.
# This directory will be used by users unless they have their own 'directory' defined.
# Default is '.' (current directory).
directory: /data

# The default permissions for users. This is a case insensitive option. Possible
# permissions: C (Create), R (Read), U (Update), D (Delete). You can combine multiple
# permissions. For example, to allow to read and create, set "RC". Default is "R".
permissions: CRUD // [!code highlight]

# The default permissions rules for users. Default is none. Rules are applied
# from last to first, that is, the first rule that matches the request, starting
# from the end, will be applied to the request.
rules: []

# The behavior of redefining the rules for users. It can be:
# - overwrite: when a user has rules defined, these will overwrite any global
#   rules already defined. That is, the global rules are not applicable to the
#   user.
# - append: when a user has rules defined, these will be appended to the global
#   rules already defined. That is, for this user, their own specific rules will
#   be checked first, and then the global rules.
# Default is 'overwrite'.
rulesBehavior: overwrite

# Logging configuration
log:
  # Logging format ('console', 'json'). Default is 'console'.
  format: console
  # Enable or disable colors. Default is 'true'. Only applied if format is 'console'.
  colors: true
  # Logging outputs. You can have more than one output. Default is only 'stderr'.
  outputs:
  - stderr

# CORS configuration
cors:
  # Whether or not CORS configuration should be applied. Default is 'false'.
  enabled: true
  credentials: true
  allowed_headers:
    - Depth
  allowed_hosts:
    - http://localhost:8080
  allowed_methods:
    - GET
  exposed_headers:
    - Content-Length
    - Content-Range

# The list of users. If the list is empty, then there will be no authentication.
# Otherwise, basic authentication will automatically be configured.
#
# If you're delegating the authentication to a different service, you can proxy
# the username using basic authentication, and then disable webdav's password
# check using the option:
#
# noPassword: true
users:
  - username: "{env}ENV_USERNAME" // [!code highlight]
    password: "{env}ENV_PASSWORD" // [!code highlight]
```

Repository에 나와 있는 config 예시를 조금 수정했습니다. user를 환경변수로 관리하고 기본 권한을 Read만 할 수 있는 것에서 CRUD 모두 가능하도록 설정했습니다. 리포지터리의 설명서에서는 이렇게 config파일을 설정하고 아래의 명령을 실행하면 서버가 띄워진다고 말하고 있습니다.

```bash
docker run \
 -p 6060:6060 \
 -v $(pwd)/config.yml:/config.yml:ro \
 -v $(pwd)/data:/data \
 ghcr.io/hacdias/webdav -c /config.yml
```

명령을 살펴보면

1. 6060으로 컨테이너 내부와 hostOS의 port를 연결
2. bind mount를 사용해서 현재 shell이 위치하고 있는 config.yml파일을 컨테이너 내부로 read only로 mount
3. bind mount로 data folder를 mount
4. 실행되는 webdav process에 `-c `옵션을 주어 2번에서 mount 한 config.yml파일을 사용하고 있습니다.

미리 `data` 폴더와 `config.yml` 파일을 생성해 주고 위의 커맨드를 입력하면 서버를 띄울 수 있습니다.
이제 이걸 원격 서버에서 실행하면 되겠죠.

저는 서비스를 더 편하게 배포하고 관리하기 위해 제 OCI 인스턴스에 [Coolify](https://coolify.io/) PaaS를 사용하고 있습니다.

## Coolify

Coolify는 오픈 소스 및 셀프 호스팅 가능한 플랫폼으로, Heroku, Netlify, Vercel의 대안으로 사용됩니다. 사용자는 자신의 서버(VPS, Raspberry Pi, EC2, DigitalOcean, Linode, Hetzner 등) 다양한 서버에 리소스를 배포할 수 있습니다. **Docker와 호환되는 모든 서비스를 배포할 수 있으며**, 많은 원클릭 서비스가 제공됩니다.

네, Docker와 호환되는 모든 서비스를 배포할 수 있습니다. WebDAV Server를 위한 모든 설정은 위에서 다루었으니 이걸 이제 coolify에 적용하면 됩니다.

coolify기본 셋팅이나 설정은 아래 링크를 참고하면 도움이 되실겁니다.

```embed
title: "Coolify Crash Course | Self Host 101 | Secure Set up"
image: "https://i.ytimg.com/vi/taJlPG82Ucw/maxresdefault.jpg"
description: "In this video CJ shows you what Coolify is, what it does, how to choose a server to deploy it to, how to lock it down with https, how to deploy several types…"
url: "https://www.youtube.com/watch?v=taJlPG82Ucw&t=1005s"
```

모든 설정을 마친 뒤 Project를 만들고 Application을 만들어주세요. Resources탭에서 New를 누르면 아래와 같은 화면이 나옵니다.
![[ys8g34.png]]
저희는 간단하게 Dockerfile을 통해 배포할 것이니 Docker Based의 Dockerfile을 선택해 줍니다.
server를 선택하면 Dockerfile을 작성하는 폼이 나옵니다. 이전에 저희가 작성해 둔 Dockerfile을 넣어줍니다.
![[13v9b0.png]]

그러면 프로젝트 설정하는 창이 나오는데 저희가 여기에서 설정할 것은 크게 두 가지입니다.

1. [[#bind mount 설정]]
2. [[# env user 설정]]

### bind mount 설정

bind mount 설정은 Persistant Storage Tab에서 할 수 있습니다.

`+Add`버튼을 누르면 아래와 같은 폼이 나오는데 저희는 **Volume Mount**에 설정을 입력하고 Add합니다. Source Path가 hostOS의 파일 경로이고 Destination Path가 Container내부 파일 경로입니다.
![[egg372.png]]
이런 과정으로 처음 bind mount 했던 두 가지(config.yml, data)를 바인딩하면 아래와 같이 설정됩니다.

> [!note]
> 굳이 bind mount를 사용하지 않아도 됩니다. 익명 볼륨, 네임드 볼륨을 사용해도 무관합니다. 관련 설정은 [이 링크](https://coolify.io/docs/knowledge-base/persistent-storage)를 참고해주세요

![[n6j98d.png]]
다음은 user env 설정입니다.

### env user 설정

Environment Variables 탭에서 `config.yml`에서 설정했던 env이름으로 key-value쌍을 만들어 추가해 줍니다.

![[3r0hyb.png]]

이렇게 설정하고 이제 Deploy를 누르면 배포됩니다. 이제 남은건 저희 Zotero어플리케이션과 File Sync를 하는 것뿐입니다.

### Zotero File Sync

![[cc5xvn.png]]

Zotero의 설정 창에 들어가 동기화 탭에 들어가면 파일 동기화 부분이 있습니다. 여기에서 동기화 도구를 `WebDAV`로 바꿔주고 폼을 입력해 줍니다.

- URL: coolify에서 domain을 설정했다면 생성된 domain을 입력해 줍니다. 아니라면 서버의 IP를 입력하고 port를 config에 설정한 대로 6065로 입력해 줍니다. 아래는 입력 예시입니다.
  - example.jinmu.me
  - 123.123.123.123:6065
- 사용자명: 환경변수 `ENV_USERNAME`의 값
- 비밀번호: 환경변수 `ENV_PASSWORD`의 값

이렇게 설정하고 zotero application 오른쪽 위에 있는 동기화 버튼(🔄)을 누르면 서버와WebDAV 프로토콜을 지켜서 sync를 시작합니다. 이렇게 우리만의zoteroo서지 관리 서버를 갖게 되었습니다.
이제 아이패드에 하이라이팅하거나 메모,캡처 등을을 남겨 놓아도인터넷만 연결되어 있다면 모든 기기와동기화할 수 있습니다.

## Outro

단어 외우겠다고 서버 구축을 시작하였지만, 꽤 재미도 있고 아이패드랑 휴대전화, 맥북에서 전부 동기화가 되는 걸 보니 괜스레 뿌듯해지네요 :)

이 글을 읽는 여러분들께도 도움이 되었으면 좋겠습니다. 혹시 서버를 구성하다가 중간에 막히셨다면 댓글이나 메일을 통해 물어보시면 답변드리겠습니다. 감사합니다.
