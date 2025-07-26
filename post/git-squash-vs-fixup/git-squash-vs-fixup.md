---
title: Git Squash vs Fixup
date: 2025-07-11T01:13:41.081Z
draft: false
tags:
  - git
lastmod: 2025-07-12T00:01:55+09:00
category: develop
bannerOnlyText: true
---

## Intro

`git`을 사용하는 워크플로우에서 remote에 push하기 이전 커밋들을 정리할 때 Interactive Rebase(`git rebase -i`)기능을 자주사용합니다. 지금까지 한 작업들을 돌아보기에도 좋고 커밋이 정확히 의미있는 단위로 묶어졌는지 확인할 때도 좋죠. 이때, 여러 커밋을 하나로 합칠 때 저는 `squash`만 사용했었습니다. 그것만 있는줄 알았거든요. 😅 그런데 이번에 `lazygit`을 사용하면서 `fixup` 커맨드를 발견했습니다. 이게 뭔가 했더니 제가 `squash`를 사용하면서 원하던 기능 이었습니다. 오늘은 Interactive Rebase시에 사용할 수 있는 명령어인 `squash`와 `fixup`을 비교하면서 어떨때 사용해야하는지 알아 보겠습니다.

![[git-squash-vs-fixup-1752198125071.webp]]

`git rebase -i`를 입력했을 때 나오는 `interactive`화면입니다. 사용하신 분들은 아시다시피 커밋의 이름앞에 command을 적어 해당 커밋에서 원하는 작업을 할 수 있습니다. `squash`와 `fixup`은 이때 사용할 수 있습니다.

## `squash` (또는 `s`)

- **기능:** 해당 커밋을 이전 커밋과 합칩니다.
- **커밋 메시지 처리:** 합쳐지는 모든 커밋의 메시지를 편집할 수 있도록 에디터가 열립니다. 즉, **이전 커밋 메시지와 현재 커밋 메시지를 모두 보여주며, 사용자가 이들을 조합하거나 새로운 메시지를 작성**할 수 있도록 합니다.
- **주요 사용 시점:** 여러 개의 작은 기능 추가 커밋이나 리팩토링 커밋들을 하나의 의미 있는 큰 기능 커밋으로 만들고 싶을 때, 또는 여러 수정 커밋들을 하나의 깔끔한 커밋으로 정리하면서 그 내용을 통합된 하나의 메시지로 남기고 싶을 때 유용합니다.

|  |  |
|:-:|:-:|
| ![[git-squash-vs-fixup-1752198570627.webp]] | ![[git-squash-vs-fixup-1752199918988.webp]] |
## `fixup` (또는 `f`)

- **기능:** 해당 커밋을 이전 커밋과 합칩니다. (`squash`와 기능은 동일합니다.)
- **커밋 메시지 처리:** 현재 커밋의 메시지를 버리고, **오직 이전 커밋의 메시지만을 유지**합니다. 별도의 에디터가 열리지 않고 자동으로 메시지가 처리됩니다.
- **주요 사용 시점:** 오타 수정, 아주 작은 버그 수정, 이전 커밋의 사소한 변경 누락 등과 같이 이전 커밋의 내용을 보완하거나 수정하는 커밋을 만들 때 사용합니다. 이 경우, 새로운 커밋 메시지가 불필요하며 원래 커밋의 메시지가 변경 사항을 잘 설명하는 경우에 적합합니다. `git commit --fixup <commit-hash>` 명령어로 커밋을 생성하면 `fixup!` 접두사가 붙은 커밋 메시지가 자동으로 생성되어 `git rebase --autosquash`와 함께 사용 시 매우 편리합니다.

## 예시를 통해 알아보기

만약 이러한 아직 remote에 push되지 않은 커밋들이 있을 때

|  |  |
|:-:|:-:|
| ![[git-squash-vs-fixup-1752199918988.webp]] | ![[git-squash-vs-fixup-1752199951224.webp]] |

저는 lazygit의 config를 수정하고 커밋 `3f05b2a` 에 합치고 싶습니다.

```text
* 3fa9966 - (HEAD -> main) remove: zsh-completions (46 seconds ago) <JinmuGo>
* 6cfcc67 - fix(git): author name (6 minutes ago) <JinmuGo>
* 52e1173 - fix: a wezterm config typo (14 minutes ago) <JinMuGo>
* 3f05b2a - feat: add lazygit config (14 minutes ago) <JinMuGo> // [!code highlight]
* e9f4306 - refactor: cleanup zsh config (14 minutes ago) <JinMuGo>
* a7020a2 - refactor: tmux config (15 minutes ago) <JinMuGo>
* 76add98 - remove: catppuccin-tmux module (24 hours ago) <JinMuGo>
```

이때 저는 수정 파일을 stage하고 `git commit --fixup  3f05b2a`하게 되면

![[git-squash-vs-fixup-1752200222658.webp]]

이렇게 `fixup! feat: add lazygit config`라는 이름으로 커밋이 하나 생겨나게 됩니다. 여기에서 볼 수 있듯이 `--fixup`은 커밋 메시지를 손쉽게 `fixup`기능을 사용하기 위해 있는 단순한 템플릿 커맨드이고 저희가 직접 `fixup!`접두어를 추가해서 만들 수 도 있겠죠?

이렇게 fixup! 접두사를 가진 커밋은 git rebase interactive시에 해당 커밋으로 합쳐지게 됩니다. 이때 `git rebase -i --autosquash`를 입력하게 되면, 커밋 목록을 자동으로 분석하여 `fixup!` 또는 `squash!` 접두사를 가진 커밋들을 해당 대상 커밋 바로 아래로 이동시키고, `pick` 대신 **`fixup` 또는 `squash` 액션으로 변경**해줍니다.

![[git-squash-vs-fixup-1752200853821.webp]]

이렇게 말이죠! 그리고 저장하고 나오면
![[git-squash-vs-fixup-1752201099133.webp]]

이렇게 제가 수정한 커밋이 깔끔하게 합쳐진 것을 볼 수 있습니다.

## 요약 비교

| 특징            | `squash`                                       | `fixup`                                         |
| --------------- | ---------------------------------------------- | ----------------------------------------------- |
| **합치기**      | 이전 커밋과 합침                               | 이전 커밋과 합침                                |
| **메시지 처리** | 합쳐지는 모든 커밋 메시지를 통합하여 편집 가능 | 현재 커밋 메시지 버리고 이전 커밋 메시지 유지   |
| **에디터**      | 열림                                           | 열리지 않음                                     |
| **주요 용도**   | 여러 커밋의 내용과 메시지를 의미 있게 통합     | 이전 커밋의 사소한 수정/보완 (메시지 필요 없음) |

> [!note] 💡 참고
> Interactive rebase에서는 `edit`(커밋을 수정), `drop`(커밋 삭제), `reword`(메시지만 수정) 등의 명령어도 사용할 수 있습니다.

## 유용한 팁들

1. `--autosquash`를 기본 설정으로 만들기
   - 매번 `--autosquash`를 입력하기 번거롭다면 Git 설정으로 기본 동작으로 만들 수 있습니다
   - `git config --global rebase.autoSquash true`
2. 빠른 fixup 커밋을 위한 alias
   - `git config --global alias.fixup 'commit --fixup'`
   - 이제 `git fixup <commit-hash>`로 간단하게 사용할 수 있어요!

## Outro

이제 `squash`와 `fixup`의 차이를 명확히 알게 되었습니다. 예전에는 모든 커밋을 합칠 때 `squash`만 사용했었는데, 사실 대부분의 경우에는 `fixup`이 더 적합했던 것 같습니다. 특히 오타 수정이나 작은 버그 픽스 같은 경우에는 굳이 커밋 메시지를 다시 편집할 필요가 없었거든요.

그리고 `git commit --fixup`과 `git rebase --autosquash`의 조합은 정말 강력합니다. 작업하면서 실수를 발견했을 때 즉시 `fixup` 커밋을 만들어두고, 나중에 한 번에 정리하는 워크플로우를 사용하면 커밋 히스토리를 훨씬 깔끔하게 유지할 수 있을 것 같아요.

앞으로는 상황에 맞게 `squash`와 `fixup`을 구분해서 사용해 보려고 합니다. 여러분도 커밋을 정리할 때 이 두 명령어의 차이를 고려해서 더 효율적인 Git 워크플로우를 만들어보시길 바랍니다! 🚀
