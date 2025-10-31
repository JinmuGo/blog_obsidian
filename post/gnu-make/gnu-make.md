---
title: 당신의 프로젝트에 Makefile이 필요한 이유
date: 2025-08-10T04:50:33.929Z
tags:
  - Makefile
  - make
category: develop
draft: true
lastmod: 2025-08-19T16:57:14+09:00
layout: 
series: 
seriesOrder: 
banner: 
---
## Intro

예전에 구름톤에서 협업하던 프론트엔드 팀원분이 제가 `Makefile`을 사용하는 것을 보고 무척 신기해하셨습니다. `make`를 사용하는 개발자는 처음 본다는 말에 곰곰이 생각해보니, 제 주변에도 `Makefile`을 적극적으로 활용하는 개발자는 많지 않더군요.

대부분은 반복적인 작업을 위해 셸 스크립트(`.sh`)를 작성하거나, JS 프로젝트의 경우 `package.json`의 `scripts`에 명령어를 등록해 사용하죠. 하지만 프로젝트가 복잡해지면 이런 스크립트들은 점점 길고 관리하기 어려워집니다. 아래와 같은 밈처럼 말이죠.

– What cats say when they’re hungry?  
	– Meow!
– What dogs say when they smell danger?
	– Woof-woof!
 
 – And what does Bob say when he’s deploying project?  
	 – `ansible-playbook -i inventory/production --tags “deploy” app-server.yml -vvv -become-user=app --extra-vars=extra.txt --vault-passwordfile="~/.ansible/vault.txt"`

출처:  https://makefile.site/

이 글에서는 바로 이런 복잡하고 기억하기 어려운 명령어들을 **`Makefile`을 통해 어떻게 간단하고 우아하게 관리할 수 있는지**, 그리고 왜 여러분의 프로젝트에 Makefile이 필요한지 이야기해보려 합니다.

## Makefile에 대해

`Makefile`을 이야기하려면 먼저 `make`라는 도구를 알아야 합니다. `make`는 1976년에 처음 등장한 아주 오래된 도구로, 본래 C언어 소스 코드를 컴파일하고 빌드하는 과정을 자동화하기 위해 만들어졌습니다. 특정 파일이 변경되었을 때만 딱 필요한 부분만 다시 컴파일하는 '의존성 관리' 기능이 핵심이었죠.

하지만 세월이 흘러, 개발자들은 `make`을 다른 곳에도 활용하기 시작했습니다. 바로 **반복적인 명령어들을 미리 정의해두고 짧은 명령어로 실행하는 '작업 실행기(Task Runner)'** 로서의 역할입니다.

다시 말해, Makefile에 우리 프로젝트에서 자주 사용하는 명령어 레시피 북으로 사용하는 겁니다.

- `docker-compose up -d --build` 같은 복잡한 도커 실행 명령어는 `make run` 이라는 레시피로,
- `go test -v ./...` 같은 테스트 명령어는 `make test` 라는 레시피로,
- Intro에서 봤던 명령어는 `make deploy` 라는 레시피로 만들어 둘 수 있죠.

```makefile
deploy: ansible-playbook -i inventory/production --tags “deploy” app-server.yml -vvv -become-user=app --extra-vars=extra.txt --vault-password-file="~/.ansible/vault.txt"
```

이렇게 하면 팀의 누구든지 길고 복잡한 명령어를 외우거나 찾아볼 필요 없이, 간단하고 통일된 `make` 명령어만으로 프로젝트의 주요 작업을 수행할 수 있게 됩니다.