---
title: Amazon Q Developer CLI 간단 체험기
date: 2025-06-04T13:25:20.437Z
draft: false
layout: PostBannerX
summary:
tags:
  - AmazonQCLI
  - AWS
lastmod: 2025-06-05T13:34:57+09:00
category: aws
---

AWSKRUG에서 Amazon Q Developer로 간단한 게임 만드는 이벤트를 한다고 알려줘서 참여해보았습니다.
![[image-67.webp]]

참여 방법은 간단합니다.

Amazon Q CLI로 게임을 만들고 어떻게 만들었는지 블로그를 쓰거나 비디오를 만들어서 관련 링크를 Form에 제출하면 됩니다.

> [!info] 참여방법
>
> **STEP 1** : Sign up to an AWS Builder ID and claim your unique community.aws username from [this link.](https://community.aws/builderid?trk=b085178b-f0cb-447b-b32d-bd0641720467&sc_channel=el) If you need help or network with other fellow builders, join the discord [server from this link](https://discord.gg/KNC9JQAfKT).
>
> **STEP 2** : [Install Amazon Q CLI](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html) in your machine. Here are two guides from Ricardo Sueiras on how you can install in [Linux](https://community.aws/content/2ulGwNwLFj5grS8hXJBMCN78Qwl/the-essential-guide-to-installing-amazon-q-developer-cli-on-linux?trk=6f6cb092-f1ba-456b-8644-73ed7ccbd567&sc_channel=el_) and in [Windows](https://community.aws/content/2v5PptEEYT2y0lRmZbFQtECA66M/the-essential-guide-to-installing-amazon-q-developer-cli-on-windows?trk=e07eca93-fa2f-4351-b567-f293b83eb635&sc_channel=el_). Install [PyGame](https://www.pygame.org/wiki/GettingStarted) or some other gaming library on your laptop.
>
> **STEP 3** :Start a chat session with Amazon Q CLI and build a game with just prompts in the chat. Try to be innovative as possible with your game so you explore the possibilities of Amazon Q CLI.
>
> **STEP 4** : Write a blog about what you’ve built or create a video about it. Make a public post in a social media of your liking with the hashtag #AmazonQCLI. It can be in your local language, which we encourage. Optionally you can also host your code in a [github](https://github.com/) repository.
>
> **STEP 5** : [Fill this T shirt redemption form](https://pulse.aws/survey/ZO9G4AEL).

게임을 만들어야 하니 우선 Amazon Q Develop CLI를 설치해야합니다.
기본적으로 macOS과 Linux 기반 OS에서만 설치할 수 있는 것으로 보이고. Window는 WSL를 이용해야합니다. 자세한 설치방법은 아래링크에서 확인할 수 있습니다.

```embed
title: "Installing Amazon Q for command line - Amazon Q Developer"
image: "https://docs.aws.amazon.com/assets/r/images/aws_logo_light.svg"
description: "Learn how to install Amazon Q Developer for command line on different operating systems including macOS, Linux, and Windows Subsystem for Linux (WSL)."
url: "https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-installing.html"
```

설치 하고 terminal에서 `q`를 입력한후 아래와 같은 화면이 나온다면 성공입니다!
![[image-69.webp]]

## Time To Create!

설치를 완료하고 나서 게임을 만들어야 하는데 저는 Python을 자주 사용한 것도 아니고 Pygame에 대해서도 몰라서 정말 `q`에게 모든 코딩로직과 이미지 생성등을 맡겼습니다.

이전에 학교 수업에서 게임을 기획한 기획 문서가 있었는데 이걸 gpt한테 주고 영어 markdown으로 재생성해서 aws q에게 주었습니다.

Elysian-Arcana라는 이름의 게임 기획이었고 이걸 어떻게 주나 싶어서 amazon q 의 `/help`기능이 있어 살펴보았더니 `context`라는 명령이 있어서 context를 제공할 수 있었습니다.

![[image-70.webp]]

![[image-68.webp]]

그래서 q의 context에 제가 기획한 게임 기획 문서를 넣고 이 markdown문서를 기반으로 미니 게임을 만들어달라고 했습니다.

로그라이크 기반의 턴제 카드 게임을 만들어 주었네요.

![[image-71.webp]]

나름 잘 만들어주었습니다. 그래도 돌아가는 게임을 AI가 만든다는게 정말 신기하네요. 디테일한 수정사항이나 게임성외 많은 것들은 여전히 부족하지만 그래도 정말 바이브 코딩으로만 만든 결과물이 이정도 라는게 신기합니다.

![[image-72.webp]]
그래도 나름 게임성을 챙기기 위해 한 스테이지를 클리어하면 카드를 고를 수 있게 만들어 주네요.

오 스테이지가 지나니 제 캐릭터들이 레벨업을 하네요. 마나통도 늘어나고 체력도 늘어났습니다. 그리고 이제보니 상대 몬스터가 어떤 공격을 하는지 intent도 보여줍니다. 나름 게임편의성을 챙긴 모습도 볼 수 있습니다.

![[image-73.webp]]

이렇게 게임을 만들 수 있는건 신기하긴 하지만 아쉬운 부분도 있었습니다.
상대 몬스터의 그림을 계속해서 만들고 연결해달라고 3번 정도 요청을 했었는데 제가 prompt를 잘 작성하지 못한 탓인지 계속만들어 주지 못하더라고요 그 부분이랑 모든 카드들의 이미지도 똑같았습니다.

하지만 이정도로 발전이 된게 놀라웠습니다. 정말 저는 구현에 대한 세부사항은 아무것도 모른채 기능과 화면의 정의 같은 것들만 계속해서 요청하며 이런 수준까지 만들어 주는 것이 신기했습니다.

무료로 사용할 수 있으니 관심이 생기신다면 한번 참여해 보시는 것도 좋을 것 같습니다. 소스 코드는 아래에 첨부 해놓겠습니다.

```embed
title: "GitHub - JinmuGo/elysian_arcana_aws_q: build game with aws q"
image: "https://opengraph.githubassets.com/d0cf93f2488e20f0c8ccd3df91300d029b5f4bfc5f2e02ab63a5acd186bd0c03/JinmuGo/elysian_arcana_aws_q"
description: "build game with aws q. Contribute to JinmuGo/elysian_arcana_aws_q development by creating an account on GitHub."
url: "https://github.com/JinmuGo/elysian_arcana_aws_q"
```
