---
title: "Node VS Element"
excerpt: "Node와 Element의 차이점을 알아보자!"
date: 2024-03-21 16:00:00 +09:00
writer: 이주영
categories: javascript
tags: [javascript node element]
toc: true
toc_sticky: true
---
혼자서 ssr 방식으로 프로젝트를 진행하다보니 html에서 동적 폼을 생성하거나 특정 요소를 수정, 삭제하는 등의 작업이 필요했다.

그 과정에서 프론트 개념이 너무 부족하다는 것을 깨닫고 강의를 듣던 중 node와 element 단어가 반복적으로 언급되길래 둘의 차이점이 알고 싶었다.

그래서 찌는 `node VS element` 차이점 포스팅!

<br>

![image](https://i.pinimg.com/564x/48/85/6d/48856d1e68fd5daa19315451f571dd2c.jpg){: width="50%"}

go go..

<br>

**일단 `node`와 `element`를 설명하기 전 js를 공부한다면 자주 보게되는 `dom(document object model)`을 먼저 알아보자.**

<br>

## 1. dom(document object model)이란?

`dom(document object model)`이란 말그대로 문서 객체 모델을 의미하며 `html이나 xml과 같은 문서에 접근하기 위한 일종의 인터페이스이자 API`이다.

html의 모든 태그는 dom(document object model)의 객체인데 이 모든 객체는 javascript와 같은 스크립팅 언어에 의해 접근이 가능하며 페이지 조작이 가능하다.

이러한 dom(document object model)은 W3C의 표준 객체 모델이며 아래와 같이 계층 구조로 표현된다.

![image](https://github.com/hobbyscripterII/javascript/assets/135996109/022ec240-7b8a-482a-88bb-70077ffa5243)

## 2. node란?

![image](https://content.codecademy.com/practice/art-for-practice/dom-nodes.png)
출저: [javascript and the DOM](https://www.codecademy.com/learn/fscp-building-interactive-websites-with-javascript/modules/fecp-javascript-and-the-dom/cheatsheet)

위에서 설명한 dom의 모든 것은 node(노드)이다!

`node(노드)`는 `element(요소)`, attribute(속성), comment(주석), document(문서) 또는 기타 모든 유형의 dom 객체이다.

또한 아래에서 설명할 엘리먼트의 상위 개념이기도 하다.

노드의 구조는 document 노드가 제일 상단에 있고 다른 모든 노드가 여기에서 분기되는 트리 구조로 구성되어 있다. 노드는 루트 엘리먼트인 html 태그로부터 시작되며 가장 낮은 레벨인 text 노드까지 뻗어나간다.

노드의 구성은 크게 3가지로 나뉘는데 `1. element node(요소 노드)`, `2. text node(텍스트 노드)`, `3. attribute node(속성 노드)`로 나뉜다.

- **element node(요소 노드)** - (예) `<body>`, `<head>`, `<title>`, `<p>문단 태그</p>` ...
- **text node(텍스트 노드)** - (예) `<p>문단 태그</p>`의 `문단 태그`에 해당된다.
- **attribute node(속성 노드)** - (예) `<a href="#">링크 태그</a>`의 `href`에 해당된다.

<br>

**javascript에서는 html dom을 이용하여 node 트리에 있는 모든 node에 접근이 가능하다.**

## 3. element란?

그렇다면 element는 무엇일까?

`element(요소)`란 우리가 제일 많이 접하는 `<p>문단 태그</p>` 태그나 `<a href="#">링크 태그</a>`와 같이 내용을 포함한 시작 태그와 종료 태그까지의 범위를 element라고 한다. 또한 위에서 설명했듯이 node에 종속된 구성 요소 중 하나이다.

여기까지 읽었을 때 엘리먼트랑 태그랑 같은 맥락이 아닌가? 할 수도 있지만 아니다. 엘리먼트와 태그의 차이점은 존재한다.

> `tag(태그)`는 소스코드에서 요소의 시작과 끝을 표시하는 마크업 기호이다.
> 
> 태그는 시작 태그와 종료 태그로 나뉘며 시작 태그는 속성(attribute)와 값(value)를 가진다. 예시로 `<a href="#">링크 태그</a>`에서 `href`는 속성, `#`은 값이다.
> 
> 태그는 엘리먼트 노드이며 트리 구조를 구성한다. `<html>`은 루트 노드, `<head>`와 `<body>`는 루트 노드의 자식이다.

따라서 엘리먼트는 웹 브라우저가 문서를 구문 분석한 후 만든 문서 객체 모델의 내부 표현이며 태그는 소스코드에서 요소의 시작과 끝을 표시하는 마크업 기호이기 때문에 엄연한 차이가 있는 것이다.

## 4. 결론
node는 dom이 구성하고 있는 계층적 구조를 의미하며 element는 node에 포함되어 있다.

---
**참고 레퍼런스** <br>
[노드(node)](https://www.tcpschool.com/javascript/js_dom_node) <br>
[엘리먼트는 뭐고 태그는 또 뭐야](https://opentutorials.org/module/966/6986) <br>
[DOM tree](https://ko.javascript.info/dom-nodes) <br>
