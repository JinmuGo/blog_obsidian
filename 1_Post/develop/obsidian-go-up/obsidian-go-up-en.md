---
title: obsidian plugin 개발기
description: Obsidian Plugin을 처음 개발 해보았던 기록
publishedAt: 2024-07-07T02:40:24.461Z
draft: true
tags:
  - obsidian
mode: X
preview: /banner/develop/obsidian-go-up.jpeg
---

처음 만들어본 Obsidian 플러그인입니다.

## obsidian

아래는 플러그인의 메인 함수입니다.

```ts
private async goUp() {
		if (
			this.settings.parentProp === "" ||
			this.settings.parentProp === null
		) {
			this.alertMustSettingParentProp();
			return;
		}
		const currentFile = this.app.workspace.getActiveFile();
		if (currentFile === null) return;
		const frontmatter =
			this.app.metadataCache.getFileCache(currentFile)?.frontmatter;
		const upProperty = frontmatter?.[this.settings.parentProp];

		if (upProperty === undefined) {
			this.alertNoUpperPage();
			return;
		}

		if (Array.isArray(upProperty)) {
			goMultiPage(upProperty, this.#goPage, this.app);
		} else {
			goSinglePage(upProperty, this.#goPage);
		}
	}
```

### obsidian api

음...
