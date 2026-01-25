---
created_dtm: 2026-01-21
template_version: v20260117-001
categories:
  - 블로그
tags:
  - 글쓰기
  - hugo
  - publish-blog
title: 개발자 블로그를 시작하며 - (2) markdown 호환
date: 2026-01-21T14:37:39.101Z
lastmod: 2026-01-25T16:37:18.997Z
---
# 지난 이야기

[개발자 블로그를 시작하며 - (1) 설정](/%EA%B0%9C%EB%B0%9C%EC%9E%90%20%EB%B8%94%EB%A1%9C%EA%B7%B8%EB%A5%BC%20%EC%8B%9C%EC%9E%91%ED%95%98%EB%A9%B0%20-%20\(1\)%20%EC%84%A4%EC%A0%95)에서 Hugo + hugo-xmin theme를 사용해서 간단한 블로그를 구성하였고, 오늘은 obsidian으로 작성한 markdown 문서를 Hugo markdown 형식으로 변환하는 과정을 진행해보려한다.

# Obsidian

굳이 왜 Obisidian을 사용해서 markdown 문서를 작성하는지 의문을 가질 수도 있다. 다양한 markdown 편집기가 존재하고, 실제로 글 작성 경험에 놀라움을 주는 것도 많다. 다만, 개발자로서의 나는 지속적으로 학습하는게 중요하고, 나의 지식체계를 관리할 문서 관리 시스템이 존재한다. 이를 Obsidian을 통해 관리하고 있고, 작성한 블로그 글도 해당 시스템에 쉽게 편입시키기 위해 같은 시스템에서 글을 작성코자 했다.

# Obsidian Markdown 문법과 Hugo Markdown 문법의 괴리

Hugo는 markdown 문법을 해석해서 HTML로 랜더링하기 위해 [Goldmark](https://github.com/yuin/goldmark)라는 라이브러리를 사용한다. Goldmark는 CommonMark, Github Flavored Markdown Spec을 지원하는 Markdown 문법을 HTML로 변환시켜주는 역할을 수행한다. 그리고 우리는 Obsidian Markdown을 사용하는데, CommonMark + Github Flavored Markdown + LaTex 문법을 지원한다. 문제는 Obisidan이 Markdown 확장 문법을 지원함으로서 생기는 Hugo와의 괴리에서 발생한다.

## Obisidian Wikilink 문법

Obsidian이 지원하는 대표적인 확장 문법중에 문제를 일으키는건 Wikilink다.\
Wikilink는 파일경로를 입력하는 모든 문법을 대괄호만 사용해서 약식으로 입력할 수 있는 문법인데, 아래와 같은 형태를 지닌다.

* 내부 파일 링크 (Internal Link): Obsidian의 파일 링크를 생성할때 사용한다. wikilink와 markdown 형시을 지원한다.
* 파일에 임베드시키기(Embed Files): 내부 파일 링크처럼 wikilink 혹은 markdown 형식중에 선택하여 사용할 수 있다.
  * 내부 파일, 이미지, pdf, 심지어 검색 쿼리의 결과도 파일에 삽입할 수 있다.

```
# 내부파일 링크
[[filename]]

# alias를 사용하여 내부파일 링크
[[filename|alias-text]]

# 내부파일 임베드
![[filename]] 

# 내부 파일의 특정 Header 및 Block 임베드
![[filename#^b15695]]

# 이미지 임베드
![[filename]]

# 이미지 크기 조절 임베드
![[filename|200x100]] # 이런 문법은 Goldmark에서 지원하지 않는다.
![[filename|200]]
```

위 처럼 Obsidian에서 Wikilink 문법에 각 파일별로 지원됐으면 좋음직한 문법이 추가된 것 같고, 이를 Goldmark는 해석하지 못한다.

## 해결방안

* 1안. Goldmark에서 [wikilink 확장](https://github.com/abhinav/goldmark-wikilink) 을 지원하는데, 이걸 사용하도록 Hugo Goldmark 설정을 손본다.
  * 문제: [Obsidian Embed File](https://help.obsidian.md/embeds#Embed%20a%20note%20in%20another%20note)에서 지원하는 다양한 문법을 모두 지원하는 것 같진 않다. (확신할 수 없어 일단 패스..)
* 2안. Obsidian에서 Wikilink 형식 사용을 포기하고 Markdown 형식으로만 링크를 만들 수 있도록 한다.
  * Obsidian 설정에서 wikilink 형식을 사용할지 markdown 형식을 사용할지 선택할 수 있다. 다만, Wikilink를 사용함으로서 얻는 편리함을 포기하긴 힘들다.
* 3안. Obsidian Wikilink 문법을 Markdown 문법으로 변환하여 Hugo Markdown 문서로 저장시킨다.
  * 이 아이디어로 Obsidian 문서를 선택적으로 Hugo 문서로 변환시켜주는 플러그인이 존재한다! 이걸 적용해보자! ([obsidian-hugo-publish](https://github.com/kirito41dd/obsidian-hugo-publish))

![Pasted image 20260126005810.png](/Resources/Resources/Pasted%20image%2020260126005810.png)\
(그림 1. Obsidian 설정 > File And Link에서 wikilink 형식을 사용할지 결정할 수 있다.)

이번주는 여기까지, 다음 글은 'Obsidian Hugo Publish 적용과 기능 확장'에 대한 내용을 작성해보자.
