---
title: Monorepo기반 Next.js와 UI 패키지를 위한 Storybook Composition 설정
date: 2025-05-13T13:02:43.826Z
draft: false
layout: PostDefault
summary:
tags:
  - storybook-composition
  - monorepo
  - storybook
  - nextjs
  - chromatic
lastmod: 2025-05-14T12:28:13+09:00
category: develop
bannerOnlyText: true
---

## Intro

프로젝트를 진행하던 중 모노레포 환경에서 UI 컴포넌트를 재사용하기 위해 `packages/ui`라는 독립적인 패키지를 구성하고, 여기에 Storybook을 먼저 도입했습니다. `tsup`으로 번들링되는 이 패키지에는 버튼, 인풋, 카드 등 디자인 시스템 중심의 컴포넌트들이 포함되어 있었고, Storybook을 통해 독립적으로 확인하고 개발하도록 구성되어있었습니다.

> 👉 초기 Storybook 설정은 [Turborepo 공식 가이드](https://turborepo.com/docs/guides/tools/storybook)를 참고해 구성했습니다.

처음에는 이 구조만으로 충분하다고 생각했습니다. 그러나 점점 서비스 페이지 단위의 UI 흐름을 확인하고 싶다는 요구가 생기면서 상황이 달라졌습니다. `apps/web`은 Next.js 기반의 앱으로, 실제 페이지들은 `packages/ui`에서 정의된 컴포넌트를 조합해 만들어졌기 때문에 **"컴포넌트가 페이지에서 어떻게 동작하는지"** 를 시각적으로 확인할 필요가 있었습니다.

```text
monorepo/
├── apps/
│   ├── web/            # Next.js 앱
│   └── storybook/      # 공통 Storybook 앱 (ui 컴포넌트만 다룸)
├── packages/
    └── ui/             # tsup 기반 UI 컴포넌트 패키지
```

이를 위해, 기존의 `apps/storybook`에서 `packages/ui`의 컴포넌트를 불러오는 방식에 더해 `apps/web`의 페이지까지 함께 렌더링할 수 있지 않을까? 라는 안일한 생각으로 시작하였습니다.

## 문제 상황

초기에는 모든 스토리를 `apps/storybook`이라는 단일 Storybook 인스턴스에서 관리하고 있었습니다. 이 구조에서는 `packages/ui`의 컴포넌트를 import하여 각 스토리를 정의하면 되었고, 대부분의 use case에서 잘 작동했습니다.
하지만 이 구조를 그대로 유지한 채로 `apps/web`의 페이지를 스토리로 등록하려고 하자 여러 가지 문제가 발생했습니다.

1. `apps/web`은 Next.js 기반인데, `apps/storybook`은 일반적인 React 앱 환경이라 **Next.js 특유의 기능들 (`next/image`, `next/head` 등)** 을 사용할 수 없었습니다.
2. `apps/web`의 페이지 컴포넌트들은 다양한 Provider나 라우터 컨텍스트에 의존하고 있어서, 그냥 import만 한다고 해서 렌더링이 되는 구조가 아니었습니다.
3. 필요한 환경을 흉내 내려면 mock provider, custom decorator, webpack 설정까지 모두 세팅해야 했고, 구조가 점점 복잡해지며 본래 의도였던 Storybook의 단순성, 모듈성, 유지보수성까지 위협하게 되었습니다.

결국 단일 Storybook 인스턴스에서 모든 것을 다루려는 시도가 처음의 개발 목표와 원칙들을 어그러뜨리고 있었습니다.

## 구조 전환

이 문제를 해결하기 위해, 구조 자체를 재설계하기로 했습니다. 핵심 방향은 다음과 같습니다:

- **UI 패키지와 웹 앱을 각각 독립적인 Storybook으로 운영**
- **apps/storybook은 Storybook Composition 허브로 전환**

구체적으로는 다음과 같이 변경했습니다:

```text
monorepo/
├── apps/
│   ├── web/                 # Next.js 15 애플리케이션
│   │   └── .storybook/      # 프로젝트별 Storybook 설정
│   └── storybook/           # 통합 Storybook 호스트
├── packages/
    └── ui/                  # 공유 UI 컴포넌트
        └── .storybook/      # UI 전용 Storybook 설정
```

- `packages/ui`는 기존처럼 tsup 기반의 Storybook을 그대로 유지하되, `http://localhost:6007` 포트에서 독립적으로 실행
- `apps/web`은 Next.js 환경에 맞춘 Storybook을 별도로 구성하고, 실제 페이지나 레이아웃, context 환경을 그대로 반영한 스토리 작성 가능하게 설정 (`http://localhost:6008`)
- `apps/storybook`에서는 이 두 Storybook을 `refs` 옵션을 활용해 외부 Storybook으로 연결하는 **Storybook Composition** 허브 역할만 담당하도록 리팩토링 (`http://localhost:6006`)

이 구조 전환을 통해 각 패키지의 개발 흐름은 분리하면서도, 하나의 통합된 Storybook 뷰를 통해 전체 UI를 관리할 수 있게 되었고, 결과적으로 협업과 컴포넌트 테스트, 문서화 모두가 수월해졌습니다.

## 해결 과정

이번 섹션에서는 `apps/storybook`에서 **Storybook Composition을 어떻게 구성했는지**에 집중해 설명하려고 합니다 .`packages/ui`와 `apps/web` 각각 Storybook 설정이 올바르게 되었다고 가정하고 진행하겠습니다.

### Storybook Composition?

먼저, Composition이란 **하나의 Storybook에서 외부에 존재하는 다른 Storybook들을 불러와 하나의 UI로 통합해 보여주는 기능**입니다. 각 Storybook은 독립적으로 **실행되고** 있어야 합니다. 네ㅁ, 실행되고 있어야 해요. production환경에서 빌드된 `storybook-static` 폴더를 제공해주는 서버가 있어야 한다는 의미입니다. 저희 서비스에서는 이 역할을 `chromatic`이라는 서비스가 담당합니다.

```ts
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  stories: ["../src/README.mdx"],
  framework: {
    name: "@storybook/react-vite",
    options: {},
  },
  addons: ["@storybook/addon-essentials"],
  refs: (config, { configType }) => {
    if (configType === "DEVELOPMENT") {
      return {
        web: {
          title: "Web Development",
          url: "http://localhost:6007",
        },
        ui: {
          title: "UI Development",
          url: "http://localhost:6008",
        },
      };
    }
    return {
      web: {
        title: "web",
        url: // web package storybook 배포 url
      },
      ui: {
        title: "UI",
        url: // ui package storybook 배포 url
      },
    };
  },
};
export default config;
```

### 실행 방식

```bash
# 터미널 1: UI Storybook
pnpm --filter packages/ui storybook

# 터미널 2: Web Storybook
pnpm --filter apps/web storybook

# 터미널 3: Composition Storybook
pnpm --filter apps/storybook storybook
```

이렇게 세 개의 스토리북을 병렬로 실행하면, `apps/storybook`에서 통합된 UI를 한 번에 확인할 수 있습니다. 각 패키지의 개발 흐름은 그대로 유지하면서도, 전체 프로젝트의 UI 흐름을 확인하는 중앙 Storybook 허브가 된 셈이죠.
_물론 turborepo나 다른 cli tool로도 병렬실행해서 사용할 수 있습니다._

### Storyboook Ref?

`refs`는 **외부에 존재하는 Storybook 인스턴스를 현재 Storybook의 왼쪽 탐색 패널(Navigation Pane)에 통합하여 보여주는 기능**입니다. 즉, 이 옵션은 *로컬이든 원격이든 Storybook의 JSON 메타 정보를 불러와 구성 요소 목록을 병합*합니다.

- `title`: 탐색 패널에 표시될 이름입니다.
- `url`: 외부 Storybook 인스턴스의 URL입니다. 보통은 해당 패키지의 `storybook dev` 주소(`localhost:포트`)이거나, 배포된 정적 Storybook의 주소입니다.

위 예시는 개발 중일 때 `ui`와 `web`의 Storybook을 로컬에서 동시에 실행하며 Composition을 확인하는 구조입니다.

> [!seealso]
> 🧠 참고로, 내부적으로는 `url` 뒤에 `/index.json`로 접근하여 메타 정보를 파싱합니다.  
> 즉, `http://localhost:6006/index.json`로 접근 가능한 상태여야 Composition이 동작합니다.

```text
apps/storybook (ref viewer)
├── refs/
    ├── ui (http://localhost:6006)
    └── web (http://localhost:6007)
```

이 구조는 Storybook이 실제 컴포넌트 코드를 불러오는 것이 아니라, **iframe으로 해당 스토리를 “프록시처럼" 보여주는 구조**이기 때문에 다음과 같은 특징이 있습니다:

| 항목                     | 설명                                                                        |
| ------------------------ | --------------------------------------------------------------------------- |
| 🔧 빌드 방식 분리        | 각 패키지의 Storybook을 독립적으로 빌드 및 배포 가능                        |
| 📦 의존성 격리           | `apps/storybook`은 실제 코드나 컴포넌트 의존성이 없음                       |
| 🚀 CI 배포 확장 용이     | 각 패키지의 Storybook을 별도로 호스팅하고, Composition은 정적으로 참조 가능 |
| 👀 페이지 초기 로딩 빠름 | 초기 Composition Storybook을 빠르게 로드                                    |

이렇게 기본적으로 Storybook Composition을 설정할 수 있었습니다. 하지만, **예상치 못한 문제가 하나 발생**했습니다.

### 로컬 환경에서 CORS 에러 발생

Storybook Composition을 설정하고 `apps/storybook`을 실행했을 때, 처음엔 아무런 문제가 없어 보였습니다. 그런데 `refs`로 연결된 외부 Storybook(`packages/ui`, `apps/web`)이 아직 실행되지 않았거나 로딩 중일 경우, 아래와 같은 **CORS 에러**가 발생하며 Storybook이 제대로 표시되지 않았습니다.

![[image-1.webp]]

이 문제는 [공식 GitHub 이슈 #17696](https://github.com/storybookjs/storybook/issues/17696)에서도 논의된 바 있는데요. 핵심은 다음과 같습니다:

> Storybook Composition에서는 `refs.url`로 지정된 외부 Storybook의 `/index.json`에 접근하여 메타 정보를 파싱하는데, 해당 서버가 아직 **시작되지 않았거나 포트가 비어 있는 상태**일 경우, 브라우저가 **HTML이 아닌 잘못된 응답 혹은 오류 페이지**를 받아 처리하게 됩니다.  
> 이때 브라우저는 이를 **CORS 정책 위반**으로 간주해 차단하게 됩니다.

### 해결 방법: `wait-on`으로 refs 대상이 준비될 때까지 기다리기

이 문제를 해결하기 위해, `apps/storybook`에서 [`wait-on`](https://www.npmjs.com/package/wait-on)이라는 library 사용했습니다. 이 라이브러리는 특정 포트, URL, 파일 등이 **정상적으로 열릴 때까지 대기**한 후에 다음 명령을 실행할 수 있게 해줍니다.

```json
"scripts": {
  "storybook": "wait-on http://localhost:6006 http://localhost:6007 && start-storybook -p 6008"
}
```

`turbo run storybook` 등으로 여러 패키지를 병렬 실행할 경우 특히 더 유용하게 사용할 수 있습니다.

## 배포 환경 Composition 구성

로컬 개발 환경에서는 각 패키지의 Storybook 인스턴스를 `localhost` 기반 URL로 참조했지만, 배포 환경에서는 이 방식이 통하지 않습니다. 특히 팀원들이 Storybook을 언제 어디서나 접근할 수 있게 하려면, **정적 호스팅된 Storybook을 기준으로 Composition을 구성해야** 합니다.
이를 위해 저는 [**Chromatic**](https://www.chromatic.com/)를 활용했습니다.

### 각 패키지를 개별 Storybook 프로젝트로 등록

각 패키지(`packages/ui`, `apps/web`, `apps/storybook`)를 독립된 Chromatic 프로젝트로 등록해 관리했습니다.

이렇게 하면 다음과 같은 주소를 얻게 됩니다:

- `packages/ui`: `https://<project-id>.chromatic.com`
- `apps/web`: `https://<project-id>.chromatic.com`
- `apps/storybook`: `https://<project-id>.chromatic.com`

각 프로젝트는 고유한 URL을 가지며, 이를 통해 `apps/storybook`에서 다른 Storybook들을 참조하게 됩니다.

### Permalinks를 활용한 안정적인 refs 구성

Chromatic의 기본 배포 URL은 빌드 ID에 따라 바뀌기 때문에, 매번 새로운 주소를 `refs`에 반영해야 하는 번거로움이 있습니다.  
이 문제를 해결하기 위해 저는 [**Chromatic의 Permalink 기능**](https://www.chromatic.com/docs/permalinks)을 사용했습니다.

> Permalinks는 특정 빌드에 고정된 URL이 아니라, **브랜치 기준(예: `main`)으로 최신 빌드를 자동 반영하는 고정 주소**입니다.

```ts
// apps/storybook/.storybook/main.ts (production)
refs: {
  ui: {
    title: 'UI Components',
    url: 'https://<branch>--<appid>.chromatic.com',
  },
  web: {
    title: 'Web App',
    url: 'https://<branch>--<appid>.chromatic.com',
  },
}
```

이렇게 하면, `main` 브랜치에 머지될 때마다 최신 Storybook 빌드가 자동 반영되고,  
`apps/storybook`에서 별도의 수동 변경 없이 항상 최신 상태로 UI를 미리볼 수 있게 됩니다.

![[image-5.webp]]

## Outro

처음에는 그냥 `packages/ui`에만 Storybook을 적용하면 충분하다고 생각했습니다. 실제 서비스에서 사용하는 페이지들을 굳이 Storybook으로 보지 않아도 될 줄 알았거든요. 그런데 프로젝트가 커지고, 컴포넌트가 어디서 어떻게 쓰이는지 확인하려고 할 때마다 코드 열고 브라우저 띄우고 로그 찍고... 생각보다 번거로운 일들이 많아졌습니다. 그리고 저만 보는 것이 아닌 다른 팀원들도 볼 경우가 생겨서 더더욱 필요성을 절감하였습니다.
그렇게 호기롭게 `apps/web`까지 Storybook을 확장해보려 했지만, 생각보다 쉽지 않았습니다. 게다가 `apps/storybook` 하나에서 모든 걸 다 처리하려 하니, 점점 구조도 복잡해지고 유지보수도 어려워졌죠.
그래서 각 패키지별로 Storybook을 따로 두고, `apps/storybook`은 그걸 조립만 하는 역할로 두기로 방향을 바꿨습니다.

그 결과, 지금은:

- 각 패키지는 독립적으로 개발되고
- 하나의 Storybook 허브에서 전체 UI를 빠르게 살펴볼 수 있으며
- 팀원들도 링크 하나만 공유하면 쉽게 컴포넌트를 확인할 수 있게 되었습니다.

기술적으로도 만족스럽지만, 무엇보다 협업할 때 훨씬 효율이 좋아졌습니다. Storybook은 단순한 문서화 도구가 아니라, 팀 전체의 커뮤니케이션을 도와주는 좋은 수단이라는 걸 다시 한번 느끼게 된 경험이었습니다.

## Reference

```embed
title: "Storybook Composition | Storybook docs"
image: "https://storybook.js.org/opengraph-image.jpg?841db310ba5a3a1e"
description: "Storybook is a frontend workshop for building UI components and pages in isolation. Thousands of teams use it for UI development, testing, and documentation. It’s open source and free."
url: "https://storybook.js.org/docs/sharing/storybook-composition"
```
