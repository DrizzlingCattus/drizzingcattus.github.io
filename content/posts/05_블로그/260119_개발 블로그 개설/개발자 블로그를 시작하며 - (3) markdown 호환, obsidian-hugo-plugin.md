---
created_dtm: 2026-02-02
template_version: v20260117-001
categories:
  - 블로그
tags:
  - 글쓰기
  - hugo
  - publish-blog
title: 개발자 블로그를 시작하며 - (3) markdown 호환, obsidian-hugo-plugin
date: 2026-02-02T13:54:32.212Z
lastmod: 2026-02-02T15:29:03.349Z
---
# 지난 이야기

지난번에는 '[개발자 블로그를 시작하며 - (2) markdown 호환](/%EA%B0%9C%EB%B0%9C%EC%9E%90%20%EB%B8%94%EB%A1%9C%EA%B7%B8%EB%A5%BC%20%EC%8B%9C%EC%9E%91%ED%95%98%EB%A9%B0%20-%20\(2\)%20markdown%20%ED%98%B8%ED%99%98)'에서 obsidian과 hugo에서 지원하는 markdown spec에 대해서 살펴봤었고, 서로 상이한 문법을 어떻게 보완하면서 같이 문서 작성/배포 흐름에 엮어 사용할지 고민하는 시간을 가졌다.

오늘은 이를 가능케하는 [obsidian-hugo-plugin](https://github.com/kirito41dd/obsidian-hugo-publish)을 적용하는 여정을 간단하게 작성해보고자 한다.

# Obsidian-Hugo-Plugin

공식 Obsidian Plugin 조회 페이지에서 찾다가 발견한 Plugin인데, 내가 원하는 기능이 바로 보였고, 사용하기도 쉬울 것 같아 적용해보자 생각이 들었다.\
![Pasted image 20260202230914.png](/Resources/Resources/Pasted%20image%2020260202230914.png)\
(그림1. Wikilink를 Markdown 형식으로 변환하는 모습을 확인할 수 있다.)

지원하는 기능은 다음처럼 심플하다.

* 파일의 Wikilink를 Markdown 형식으로 변환시킨다.
* Hugo 디렉토리 경로를 설정하여 변환한 파일을 동기화시킬 수 있다.
* 특정 디렉토리 하위에 있는 파일은 변환 및 Hugo 디렉토리 **배포에서 제외시킬 수 있다**.
* 특정 tag를 지닌 파일만 선택하여 변환 및 Hugo 디렉토리 **배포에 포함시킬 수 있다**.
* Hugo 디렉토리에서 동기화 대상에서 제외할 파일 패턴을 설정할 수 있다.

![Pasted image 20260202231822.png](/Resources/Resources/Pasted%20image%2020260202231822.png)\
(그림2. publish-blog 태그를 설정한 파일을 변환하여 site dir 하위 blog dir에 주입시킨다. 이미지 파일은 static dir에 넣는다.)

# Obsidian 문법을 부분적으로 지원하는 문제

적용한 후 대부분 문서에선 문제가 없었는데, 몇 문서가 이미지를 출력하지 못하거나 이상하게 링크를 만드는 모습을 발견했다.

![Pasted image 20260202232442.png](/Resources/Resources/Pasted%20image%2020260202232442.png)\
(그림3. 이미지 파일명을 이상하게 파싱하여 출력하지 못하는 모습.)

이는 Obsidian 확장 문법중에 아래 2가지를 지원하지 않아서 발생하는 문제다.

* Link Alias Syntax : 링크의 별칭을 설정할 수 있는 문법
  * `[[link.com|alias-text]]` -> `[alias-text](link.com)`
* Image Dimension Syntax : 이미지를 특정 크기로 출력할 수 있도록 지원하는 문법
  * `![[xxx.png|200*100]]` -> `![xxx.png](/${static_dir}/xx.png)`

대괄호 안의 모든 값(별칭과 이미지 크기도 포함)을 링크주소/이미지경로로 취급해버려서 적절히 결과를 출력하지 못하는 것인데, Plugin이 이걸 지원하도록 수정이 필요한 상황이었다.

# Obisidan Plugin 수정하기

에라 모르겠다하고 일단 Plugin 구현을 살펴봤는데, 그리 내용이 많지 않아서 금방 수정할 부분을 찾을 수 있었다. 대괄호 패턴을 Regex로 잡아서 내부의 값을 Hugo

Plugin은 [mdast](https://github.com/syntax-tree/mdast)라는 마크다운 구문 분석기를 사용하고 있는데, Abstract Syntax Tree를 만들어서 특정 패턴을 만나면 원하는 값으로 대체시키는 동작을 구현하고 있다.

* AST를 만들고 node 순회를 시작한다.
* 텍스트 타입의 node를 만날때까지 모든 node를 순회한다.
* 텍스트 타입의 node를 만나면, 대괄호 Wikilink 문법을 Regex로 잡아낸다.
* Regex로 잡아낸 텍스트를 Markdown Link로 치환시킨다.

```typescript
// AST를 만든다.
const ast = fromMarkdown(body, {
				extensions: [math(), gfmTable()],
				mdastExtensions: [
					mathFromMarkdown(),
					gfmTableFromMarkdown()
				]
			})

...(생략)...

// 특정 패턴의 문법을 변환시키기위해 Root Node부터 하위 Node까지 순회한다.
// ![[xxx.png]] -> ![xxx.png](xxx.png)
export const transform_wiki_image = (ast: Root) => {
    visit(ast, 'paragraph', function (node, index, parent) {
        transform_wiki_image_on_parent(node);
    })
    visit(ast, "tableCell", function (node, index, parent) {
        transform_wiki_image_on_parent(node);
    })
}
// [[xxx.png]] -> [xxx.png](xxx.png)
export const transform_wiki_link = (ast: Root) => {
    visit(ast, 'paragraph', function (node, index, parent) {
        transform_wiki_link_on_parent(node);
    })
    visit(ast, 'tableCell', function (node, index, parent) {
        transform_wiki_link_on_parent(node);
    })
}

...(생략)...

// 패턴일치한 텍스트를 Markdown Link로 치환시킨다.
const text = child.value.slice();
const regex = /\[\[(.*?)\]\]/g;
let match;
while ((match = regex.exec(text)) != null) { 
	// (생략)
	const v: Link = {
	    type: 'link',
	    url: encodeURI(image_url),
	    title: null,
	    children: [link_txt]
	};
	new_children.push(v);
}
```

즉, 대괄호 Wikilink 패턴의 regex를 수정하여 Link Alias와 Image Dimension 구문을 구분하도록 수정하면 끝난다.

[수정 PR](https://github.com/kirito41dd/obsidian-hugo-publish/pull/18)은 몇주전에 만들어서 올렸고, 생각보다 빨리 master 병합 및 릴리즈해주셔서 문제없이 사용중이다.\
![Pasted image 20260202233410.png](/Resources/Resources/Pasted%20image%2020260202233410.png)\
(그림4. PR을 올렸는데, 생각보다 빨리 릴리즈해주셨다.)

# Future Works

조금 더 살펴보니 내가 했던 고민을 좀더 제대로 해결하고 있는 [프로젝트](https://github.com/devidw/obsidian-to-hugo)를 발견한 것 같다.\
Obisidan의 Wikilink를 Hugo의 내부 링크 기능인 `ref` shortcode로 변환시켜주도록 기능 구현을 하신 모습이 보인다. (괜찮아 보인다...)

나중에 기회가 되면 이 프로젝트로 갈아치우는 것도 생각해봐야겠다.

# 마무리

확실히.. 난 몇 가지 확실한 기능만 지원하는 플러그인을 개인적으로 선호하는 경향이 있다.\
문제가 생기면 쉽게 기능 파악/수정이 가능하니, 내 개인 환경을 구성할땐 잔잔바리?를 섞는 내 모습을 종종 본다.

![Pasted image 20260202235559.png](/Resources/Resources/Pasted%20image%2020260202235559.png)\
(그림5. 이미지도 안깨지고, 링크 alias도 잘 적용되는 모습이다.)

오늘은 여기까지. 다음은 Hugo로 Github Page로 배포하는 방법에 대해서 소개해볼까? 하는데, 이미 도처에 관련 정보는 많으니.. 아예 다른 주제로 들고올까 한다.

+) [mdast의 실체](/mdast%20-%20%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4%20%EA%B5%AC%EB%AC%B8%20%EB%B6%84%EC%84%9D%20%EB%AA%85%EC%84%B8%20&%20%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC)는 실은 Markdown 명세를 기술한 프로젝트다. 이 명세를 참고하여 마크다운 파싱/직렬화([mdast-util-from-markdown](https://github.com/syntax-tree/mdast-util-from-markdown), [mdast-util-to-markdown](https://github.com/syntax-tree/mdast-util-to-markdown)) 라이브러리를 구현하고 있다.
