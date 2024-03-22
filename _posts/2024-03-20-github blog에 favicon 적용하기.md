---
title: "github blog에 favicon 적용하기"
excerpt: "github blog에 파비콘(favicon)을 적용해보자!"
date: 2024-03-20 19:20:00 +09:00
writer: 이주영
categories: github blog
tags: [github blog, jekyll, github, git, favicon]
toc: true
# toc_sticky: true
---
이전에 jekyll을 통해 github blog를 만들었다.
이후 내 블로그를 구경하고 있던 찰나 상단에 이 아이콘이 거슬리기 시작했다.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/43cd0e18-44ae-466b-8bf3-0079b809168a)

<br>

묘하게 존재감을 뿜어내는 이 녀석..(jekyll 테마 중 하나인 chirpy의 기본 파비콘이다)

`파비콘(favicon)`은 웹사이트 혹은 웹 페이지를 대표하는 16x16 pxcel의 작은 이미지라고 한다.

벌레를 극도로 싫어하는 편이라 이것도 거슬려서 바꾸기로 마음 먹었다.

이제 파비콘을 바꿔보자!

## 1. 파비콘(favicon) 이미지 고르기

[website planet 파비콘 생성기](https://www.websiteplanet.com/ko/webtools/favicon-generator/)를 통해 쉽게 만들 수 있다.

물론 jpg, png를 통해 자기가 선택한 사진도 파비콘으로 만들어낼 수 있다.

![apple-touch-icon-72x72](https://github.com/hobbyscripterII/csharp/assets/135996109/4d935711-f71b-4f66-a676-94924395e3a4)

나는 그나마 과일 중에 딸기를 좋아해서 딸기 아이콘으로 만들었다.

## 2. 파비콘(favicon) 생성

[real favicon generator](https://realfavicongenerator.net/)에 들어가 `Select your Favicon image`를 클릭 후 이전에 골라놨던 파비콘 이미지를 클릭한다.

18px인가 아래로면 파비콘 생성을 막기 때문에 넉넉하게 100px 이상인 파비콘 이미지를 추천한다.

로컬 이미지를 선택하면 아래 화면이 출력되는데 이 때 1번의 `Favicon package`를 누르자.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/8a210467-d5e3-41ed-98fc-41f4a63f28bd)

## 3. 파비콘(favicon) 불필요 파일 삭제

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/4ba82a32-4b0b-4cc1-b6bd-71326337db89)

<br>

다운로드 받은 폴더의 압축을 풀면 파일이 이렇게 있을건데 여기서 아래 파일 2개를 삭제한다.

- browserconfig.xml
- site.webmanifest

## 4. github blog 파비콘(favicon) 세팅

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/e8c4f20e-dfba-4a34-af64-f0cc5323aefa)

`path:\{github_username}.github.io\assets\img\favicons`에 들어가 이전에 받았던 압축 폴더 내 파일들을 싹 다 붙여넣는다.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/7cbcd84c-18c7-42d1-a16b-f12f38f9d0ba)

훨씬 보기 좋아졌다.

이제 한 단계밖에 안남았다.

<br>

`path:\{github_username}.github.io\_includes\favicons.html` 파일을 열어 아래와 같이 수정한다.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/fda5d2ce-079d-4307-adfe-11808d882913)

아까 우리가 붙여넣은 파비콘 이미지명만 잘 적어준다면 서버 재가동 시 아래와 같이 이쁘게 적용된 파비콘을 확인할 수 있다.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/38ba6129-24da-4d65-a731-6d7d24e1c4d7)

---
**참고 레퍼런스** <br>
[블로그 커스터마이징하기](https://wlqmffl0102.github.io/posts/Customizing-Blogs/)