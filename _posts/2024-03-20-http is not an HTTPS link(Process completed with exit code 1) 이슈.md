---
title: "http is not an HTTPS link(Process completed with exit code 1) 이슈"
date: 2024-03-20 21:20:00 +09:00
writer: 이주영
categories: github blog
tags: [github blog, jekyll, github, git, favicon, issue]
toc: true
# toc_sticky: true
---
깃허브 블로그 만든지도 얼마 안됐는데 벌써 트러블슈팅 블로깅 할 게 하나 생겼다!

이슈 내용은 github blog(jekyll)를 github action으로 build 시 `Process completed with exit code 1.` 에러가 발생하는 것이였다.

## 1. 로그 확인

자세한 로그는 아래와 같다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/77d5ea32-b58d-4f01-85da-64ecfbd7901c)

참고로 `Process completed with exit code 1.` 에러는 애플리케이션 혹은 이미지가 잘못된 파일을 가리키기 때문에 발생하는 에러라고 한다.

구글링해도 나랑 상황이 다 다른 케이스여서 감이 잘 안잡혔다.

```shell
* At _site/posts/나만의-github-blog-만들기(windows)/index.html:31:
  http://jekyllthemes.org is not an HTTPS link
```

그저 핵심은 위의 두 문장이라는 것만 알고 있었다.

해당 로그를 읽고 먼저 든 생각은 `_site/posts/나만의-github-blog-만들기(windows)/index.html의 31번째 줄을 확인해보자!` 였다.

그리고 확인했다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/6edfb5a7-3fd3-4415-9107-ef09af873c41)

31번째 줄에는 주석만이 날 반기고 있었다.

<br>

![image](https://i.pinimg.com/564x/15/38/7c/15387cbec5f1e74ece7aa13735ee07f9.jpg){: width="50%"}

본격적인 삽질을 시작했다.

<br>

![스크린샷 2024-03-20 195819](https://github.com/hobbyscripterII/about-play/assets/135996109/5747693d-a0ed-49fe-aa07-a5a2e8aae4e3)

파비콘 관련 포스팅을 수정한건 정말 바보짓이였다.

하지만 저 때는 파비콘 관련 포스팅 작성 후 생긴 이슈라 그 이전이면 파비콘 관련 포스팅에 힌트가 있을거라 생각했다.

그렇게 바보짓을 반복하다가 로그 창을 다시 읽어봤다.

## 2. 이슈 해결

`http://jekyllthemes.org is not an HTTPS link`

![image](https://i.pinimg.com/originals/28/5f/59/285f59a696a4fd63ed402fa4ecb6371b.gif){: width="40%"}

여기에 힌트가 있을까?
에러 발생 전 코드와 에러 발생 후 코드를 비교해봤다.

<br>

![스크린샷 2024-03-20 194445](https://github.com/hobbyscripterII/about-play/assets/135996109/9e756a19-3953-4009-9251-039bd3f57203)

에러 발생 전 코드는 단순 url을 첨부했다면 에러 발생 후 코드는 링크를 마크다운 형식으로 `[]()`에 넣어주고 있었다. 아마 `[]` 여기서 에러가 발생한 것 같다는 생각이 들었고 아래와 같이 코드를 변경해주었다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/57dd3666-3741-48d8-8bc8-02dac883f61a)

하지만 위의 코드로도 빌드 에러가 발생했다.

결국엔 아래와 같이 처음처럼 코드를 수정하니 빌드 에러가 사라졌다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/3a7f2920-fbd8-44aa-8d1a-1c783aa03324)

## 3. HTML Proofer는 왜 Http 링크를 잡아낸걸까?
`http://jekyllthemes.org`가 https 링크가 아니라는 에러가 왜 발생한걸까 생각해보니 빌드 시 HTML Proofer가 해당 에러를 잡아냈다.

`HTML Proofer`는 jekyll에서 제공하는 테스트로 `생성된 사이트의 모든 링크와 이미지가 존재하는지 확인`해주는 `테스트 스크립트`라고 한다.

그리고 구글에 'http proofer https'로 검색하니 공식 레파지토리가 있어서 확인해보니 아래와 같이 적혀져 있었다.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/6c5f3530-ebbb-44df-90ca-e79bcda7c1b5)

대충 읽어보니 마크다운 문법에 들어간 링크가 https가 아니면 에러를 발생시키는 것 같았다. 그래서 그냥 url을 직접적으로 넣었을 때랑 마크다운으로 넣었을 때랑 다르게 빌드 결과가 출력되는 것 같다.

---
**참고 레퍼런스** <br>
[jekyll HTML Proofer dosc](https://jekyllrb-ko.github.io/docs/continuous-integration/travis-ci/) <br>
[html-proofer](https://github.com/gjtorikian/html-proofer)