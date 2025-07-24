---
title: 블로그 댓글기능 적용기 With Fuma Comment
date: 2025-05-01T01:46:46.245Z
draft: false
summary: 불완전한 댓글 기능, Fuma Comment로 깔끔하게 리빌드한 후기
banner: blog-comment-feautre-with-fuma-comment.jpg
tags:
  - fuma-comment
  - postgres
  - nextatuh
  - prisma
  - uploadthing
  - github-oauth
lastmod: 2025-05-02T14:58:29+09:00
category: develop
---

> [!attention]
> 해당 글은 [fuma-comment Docs](https://fuma-comment.vercel.app/docs)과 [repository](https://github.com/fuma-nama/fuma-comment)를 참고 했습니다.

## Intro

github follow하고 있던 [velopert](https://github.com/velopert)님이 한 프로젝트를 starred 한 것을 우연히 보았습니다.
![[image2.png]]

fuma-nama님의 [fumadocs](https://github.com/fuma-nama/fumadocs)라는 프로젝트였습니다. next.js 기반 document framework인데 디자인도 유려하고 무엇보다 정말 간단하게 document를 만들 수 있더군요. 놀라웠습니다. 그렇게 이 분에 대해 염탐(?)을 하던 도중 다른 프로젝트가 눈에 들어왔습니다.
![[image3.png]]

바로 [fuma-comment](https://github.com/fuma-nama/fuma-comment)라는 프로젝트였는데요. 설명을 보니 아래와 같이 적혀 있었습니다.

> user friendly, beautiful comment area to your blog

블록에 댓글 기능을 추가할 수 있는 프로젝트였죠. 저는 이거다! 싶었습니다. 왜냐하면 최근 제 블로그 댓글 기능에 고민이 많았거든요...

![[image4.png]]

처음 댓글 기능을 만들 때는 **불편한 로그인 기능 없이 누구든 와서 댓글을 가면 좋겠다.** 라는 생각으로 최대한 OAuth, ID/PW, email, magic link등등을 제거해서 오직 **IP**로만 자신의 댓글을 CRUD할 수 있게 구현했습니다. 하지만 이 방법은 문제가 많았습니다. 😞

1. 공공장소와 같은 IP가 공유되는 상황에서는 다른 사람의 댓글을 조작할 수 있었습니다.
2. 한 사람의 IP는 고정되지 않는다. 어제 쓴 댓글인데 오늘은 수정/삭제 버튼이 안 보이는 상황이 발생할 수 있습니다.
3. 스푸핑/프록시로 악용 가능합니다. VPN/프록시로 해당 IP를 정말 간단하게 훔칠 수도 있습니다.
4. 알림/응답이 불가능합니다. IP만으로는 사용자를 식별할 수 없을 뿐더러 이메일주소를 알지도 못합니다.

이렇게 적고나니 왜 IP로만 구현했나 싶네요. 돌아보니 누구든 제 글을 보고 편하게 댓글을 남기고 갔으면 하는 마음이 더 커서 이렇게 구현한 것 같습니다. 그것이 비록 너무 쉬워서 문제였지만요. giscus도 도입을 했었지만, theme custom이 어렵고 github 아이디가 있어야 사용가능하다는점이 아쉬웠습니다.

그래서 위 프로젝트를 보고 난 뒤에 마음이 동했습니다. next-auth, better-auth, clerk auth등 다양한 Authenticator를 꽂아 사용할 수 있고 Mention기능이나 답글, 알림 기능등 다양하게 Custom가능했습니다.

아래 링크를 통해 직접 프로젝트를 테스트하거나, 댓글을 남길 수 있습니다.

```embed
title: "Fuma Comment"
image: "https://avatars.githubusercontent.com/u/104714237?v=4"
description: "A React.js library for adding comment area to anywhere, with your own backend & auth!"
url: "https://fuma-comment.vercel.app/"
```

그래서 저는 이 프로젝트를 제 블로그에 적용하기로 했습니다. 우선 지금은 간단하게 Github OAuth로 구현한뒤 나중에 magic link등으로 변환하거나 다른 방법을 생각해보려 합니다.

## Overview

client: **nextjs 15** with  **Route Handler**
database: **postgres**
ORM: **prisma**
ObjectStorageService: **uploadthing** (S3 기반)

> [!caution]
>  **Route Handler**를 사용하여 매번 새로운 요청들을 처리하기 때문에 `next.config.mjs`에 `export: output`옵션과 함께 사용할 수 없습니다.

### Library

1. `@fuma-comment/react`
2. `@fuma-comment/server`
3. `@prisma/client`
4. `prisma`
5. `uploadthing`

### Signin Flow

![[image5.png]]
Inspired From: https://medium.com/@enes9103/building-apis-and-handling-authentication-with-next-js-9d4a8b448251

### Comment Flow

![[image6.png]]

---

1. [[#DB]]
   - Prisma
   - Postgres DB
2. [[#NextJS Route Handler]]
   - NextAuth
   - uploadthing
3. [[#NextJS Client]]

순서로 구현 설명하겠습니다.

## DB

postgresDB 하나를 준비합니다. vercel에 있는 DB Provider를 사용하셔도 되고 서버에 직접 설치 또는 docker를 사용하셔서 띄우셔도 됩니다.

![[image7.png]]

저는 Coolify를 사용하고 있어서. 원격 서버에 docker를 통해 container를 띄웠습니다.
![[image8.png]]

그리고 이 DB를 원격에서 접근할 수 있도록 public 설정을 해줍니다. 방화벽의 포트를 허용하고, docker라면 port binding을 진행합니다.

이렇게 DB를 노출하는데에 성공했다면 `psql`같은 client tool로 db접근을 시도합니다. postgres의 URI는 아래와 같습니다.

```text
postgres://{POSTGRES_USERNAME}:{SERVICE_PASSWORD_POSTGRES}@{YOUR_HOST}:{POSTGRES_PORT}/{POSTGRES_DB}
```

![[image9.png]]

이제 이 postgres의 public URI를 통해 Route Handler에서 접근할 것입니다.
그전에 해야할 것이 있습니다.

`prisma/schema.prisma`경로에 링크로 이동한 뒤 코드를 붙여 넣습니다.

```embed
title: "fuma-comment/apps/docs/prisma/schema.prisma at main · fuma-nama/fuma-comment"
image: "https://repository-images.githubusercontent.com/717080536/f375c8ab-da9a-4716-973e-92b47f38244a"
description: "user friendly, beautiful comment area to your blog - fuma-nama/fuma-comment"
url: "https://github.com/fuma-nama/fuma-comment/blob/main/apps/docs/prisma/schema.prisma"
```

**`DATABASE_URL`** 환경 변수에 postgres URI를 바인딩하고 **`prisma db push`** 명령어를 통해 Prisma 스키마 파일의 현재 상태를 데이터베이스에 바로 반영합니다.

### NextJS Route Handler

`/api` 폴더 구조는 아래와 같습니다.

```text
/api
├── /auth
│   └── [...nextauth]
│       ├── options.ts
│       └── route.ts
├── /comments
│   └── [...comment]
│       └── route.ts
├── uploadthing
    ├── core.ts
    └── route.ts
```

```embed
title: "fuma-comment/apps/docs/app/api at main · fuma-nama/fuma-comment"
image: "https://repository-images.githubusercontent.com/717080536/f375c8ab-da9a-4716-973e-92b47f38244a"
description: "user friendly, beautiful comment area to your blog - fuma-nama/fuma-comment"
url: "https://github.com/fuma-nama/fuma-comment/tree/main/apps/docs/app/api"
```

경로에있는 파일들을 붙여넣고 경로들만 수정해주면됩니다.

### Github OAuth 발급

이때 NextAuth에서 Github OAuth 토큰을 사용하기 때문에

```ts
providers: [
	GithubProvider({
		clientId: env.GITHUB_ID, // 하이라이트
		clientSecret: env.GITHUB_SECRET,  // 하이라이트
		profile(profile: GithubProfile) {
			return {
				id: profile.id.toString(),
				name: profile.name || profile.login,
				email: profile.email,
				image: profile.avatar_url,
			};
		},
	}),
],
```

OAuth 를 발급받고 설정해주어야 합니다.

`profile -> Settings -> Developer Settings` 를 들어가면 아래의 화면이 나옵니다.

![[image10.png]]

여기에서 `OAuth Apps`를 클릭하여 이동해줍니다. `New OAuth app`을 누르고

개발 환경이라면 `localhost:3000`.
배포 환경이라면 배포 url 을 입력해줍니다. callback URL에는 `/api/auth/callback/github`를 추가해줍니다.

![[image12.png]]

> [!note]
> 개발과 배포 환경에서 따로 동작하게 하기 위해서 OAuth를 각각 발급받으시는 것을 추천드립니다.

그리곤 Secret을 발급 받고 Client ID와 Client Secret의 값을 환경변수 `GITHUB_ID`, `GITHUB_SECRET`에 binding해줍니다.

![[image13.png]]

### NextAuth

NextAuth관련 환경변수도 설정해주어야 합니다. 아래 링크에서 확인하실 수 있습니다.

```embed
title: "Options | NextAuth.js"
image: "https://next-auth.js.org/img/logo/logo-xs.png"
description: "Environment Variables"
url: "https://next-auth.js.org/configuration/options#nextauth_secret"
```

```text
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=
```

등으로 설정할 수 있습니다. 특히 `NEXTAUTH_SECRET`은 공식문서에서 확인할 수 있듯이 프로젝트의 root에서 `npx auth secret`을 실행하면 `NEXTAUTH_SECRET`의 alias 인 `AUTH_SECRET`으로 `.env.local`에 setting해줍니다. 이걸 이름만 바꿔서 사용하시면됩니다.

### Uploadthing

uploadthing은 댓글에서 이미지를 사용하기 위해 필요합니다. free tier 에서는 2gb를 기본으로 제공하고 있습니다. 간단한 블로그에서 사용할 만큼은 되는 것 같아서 우선 사용하고 있습니다. 만일 더 필요하게 된다면. 다른 방법을 알아봐야겠죠. S3로 호환이 되니 cloudflare R2를 사용해도 되겠네요.

아무튼, [홈페이지](https://uploadthing.com/)에서 회원가입 및 로그인을 해주신다음 `Dashboard -> API Key ` 에서 `UPLOADTHING_TOKEN`을 그대로 .env에 옮겨주시면됩니다.

---

이제 Route Handler에서의 설정은 모두 끝났습니다. 코드들은 모두 official repo에서 가져오고. 저희가 해야할 것은 env설정 밖에 없었습니다. 설정한 env는 모두 6개입니다.

```text
DATABASE_URL=
GITHUB_ID=
GITHUB_SECRET=
NEXTAUTH_URL=
NEXTAUTH_SECRET=
UPLOADTHING_TOKEN=
```

## NextJS Client

그리곤 이제 `CommentsWithAuth`라는 이름으로 컴포넌트를 만듭니다.

```ts
"use client";

import { signIn } from "next-auth/react";

import type { CommentsProps } from "@fuma-comment/react";

import { Comments } from "@fuma-comment/react";

import { createUploadThingStorage } from "@fuma-comment/react/uploadthing";

const storage = createUploadThingStorage();

export function CommentsWithAuth(props: Omit<CommentsProps, "auth">) {
  return (
    <Comments
      storage={storage}
      mention={{
        enabled: true,
      }}
      auth={{
        type: "api",
        signIn: () => void signIn("github"),
      }}
      {...props}
    />
  );
}
```

```embed
title: "fuma-comment/apps/docs/app/(demo)/page.client.tsx at main · fuma-nama/fuma-comment"
image: "https://repository-images.githubusercontent.com/717080536/f375c8ab-da9a-4716-973e-92b47f38244a"
description: "user friendly, beautiful comment area to your blog - fuma-nama/fuma-comment"
url: "https://github.com/fuma-nama/fuma-comment/blob/main/apps/docs/app/(demo)/page.client.tsx"
```

그리곤 적용하고 싶은 nextjs 페이지(`page.tsx`), 레이아웃(`layout.tsx`)또는 컴포넌트에 `import`하면 끝입니다.

```ts
<CommentsWithAuth page={slug} />
```

page props에는 각 페이지의 id또는 slug를 설정하면 페이지 별로 댓글창을 설정할 수 있습니다. 만약 같은 댓글 영역을 쓰고 싶다면 page의 값을 똑같이 맞춰주면 되겠죠.

## Outro

이렇게 `fuma-comment`를 이용해서 댓글기능을 만들어봤습니다. 이렇게 간단하게 만들어 쓰기도 좋은데 커스텀할 수 있는 방안이 굉장히 무궁무진해서 더 유용한 것 같습니다. NextAuth, prisma, postgres DB에 대한 지식이 조금만 있어도 자신이 원하는대로 커스텀할 수 있다는 것이 이 프로젝트의 장점인 것 같습니다.

멋진 프로젝트를 만들어주신 [fuma-nama](https://github.com/fuma-nama)님에게도 감사의 말씀을 드리며, 이만 글을 마치겠습니다.

> [!seealso]
> 아마 글을 작성하면서 빠진 부분이나 부족한 부분이 있을 것으로 생각됩니다. 댓글이나 메일을 남겨주시면 최대한 빠른 시일내에 답변드리겠습니다. 감사합니다.
