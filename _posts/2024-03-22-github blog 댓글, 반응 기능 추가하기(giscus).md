---
title: "github blog 댓글, 반응 기능 추가하기(giscus)"
excerpt: "giscus를 활용해 github blog에 댓글, 반응 기능을 추가해보자!"
date: 2024-03-22 11:00:00 +09:00
writer: 이주영
categories: github blog
tags: [github blog, jekyll, github, git, giscus]
toc: true
toc_sticky: true
---

블로그를 활성화시킬 생각인데 이 때 댓글/반응 기능도 추가되었으면 좋겠다싶어서 추가해봤다.

gihub blog에 댓글 기능을 사용할 수 있게 해주는 플랫폼은 크게 3가지(disqus, utterances, giscus)가 있는데 disqus는 SNS 계정으로 댓글 작성이 가능하다고 한다. 근데 굳이 기술 블로그에 SNS 계정으로 댓글을 달 필요는 없다고 생각했고, utterances를 쓰시던 분들이 giscus로 많이 갈아타길래 처음부터 giscus로 댓글 기능을 추가했다.

<br>

![image](https://i.pinimg.com/564x/48/85/6d/48856d1e68fd5daa19315451f571dd2c.jpg){: width="50%"}

go go..

## 1. repository discussions 기능 활성화

![스크린샷 2024-03-22 102340](https://github.com/hobbyscripterII/javascript/assets/135996109/9d191b53-0c8d-4efe-9861-c3913db8edb8)

giscus을 사용하기 위해선 github blog 레파지토리의 Settings - Features에 들어가 Discussions 기능을 활성화해주어야 한다.

## 2. giscus 세팅

[giscus](https://giscus.app/ko)에 들어가 `설정` 부분을 먼저 살펴보자.

`저장소`에 적힌 1, 2, 3번의 조건을 다 만족시켜야 아래와 같이 유효성 검증이 통과된다.

![스크린샷 2024-03-22 102419](https://github.com/hobbyscripterII/javascript/assets/135996109/315daa5b-3be4-435c-b24a-a273032aa7ea)

`조건`은 아래와 같다.

> 1. github blog 레파지토리는 public 이여야 한다.(근데 github blog는 원래 public 아닌가? private이여도 가능하긴 한건가?)
> 
> 2. giscus 앱이 설치되어 있어야한다. 2번의 링크를 클릭하면 아래와 같이 install 창이 뜨는데 github blog 레파지토리를 선택 후 install 버튼을 눌러주면 된다.
    ![스크린샷 2024-03-22 102026](https://github.com/hobbyscripterII/javascript/assets/135996109/12199ce7-76ee-4199-9a6b-706d61727fa1)
>
> 3. discussions 기능이 활성화되어 있어야하는데 위에서 이미 했으므로 패스하면 된다.

<br>

저장소에 github blog 레파지토리를 적었을 때 유효성 검증이 통과되면 아랫 부분을 마저 세팅해주자.

<br>

4. **페이지 ↔️ Discussions 연결** - `Discussion 제목이 페이지 <title>을 포함` 선택

5. **Discussion 카테고리** - `General` 선택

6. **기능** - `메인 포스트에 반응 남기기` 체크

giscus 세팅이 끝나고 스크롤바를 밑으로 내리면 아래와 같이 데이터가 셋팅되어 있을 것이다.

![스크린샷 2024-03-22 102730](https://github.com/hobbyscripterII/javascript/assets/135996109/ef1b3cec-fe9a-410b-98fc-bf3005b88049)

이제 위의 데이터를 통해 `_config.yml` 파일과 `_layouts\post.html` 파일만 수정해주면 된다!

## 3. _config.yml 수정

먼저 _config.yml 파일 먼저 수정하자.

![스크린샷 2024-03-22 102818](https://github.com/hobbyscripterII/javascript/assets/135996109/ec7455b9-471c-4463-82fb-ffa70d5c726f)

날 것의 yml 파일이다.

위에서 받아온 데이터를 아래와 같이 세팅하면 된다.

![스크린샷 2024-03-22 105109](https://github.com/hobbyscripterII/javascript/assets/135996109/6692c4e4-93f8-417e-a94c-c3c8120e7fc0)

## 4. _layouts\post.html 수정

그리고 마지막으로 _layouts\post.html에 들어가 아래와 같이 script 소스코드를 붙여넣어주면 끝이다!

![스크린샷 2024-03-22 105417](https://github.com/hobbyscripterII/javascript/assets/135996109/b8439b0e-79ea-4304-915b-c37c33ad3231)

## 5. 결과

![스크린샷 2024-03-22 105309](https://github.com/hobbyscripterII/javascript/assets/135996109/dfc64364-094a-4857-8fa4-323f3f626b62)

---
**참고 레퍼런스** <br>
[git blog에 댓글 기능 추가하기(jekyll, chirpy, 400, 400 error 정리](https://da-in.github.io/posts/Blog-Comments/) <br>
[블로그 댓글, 반응 추가(giscus)](https://devshjeon.github.io/78) <br>
[github blog 댓글 giscus 설정](https://jihwan98.github.io/posts/giscus-%EC%84%A4%EC%A0%95/)