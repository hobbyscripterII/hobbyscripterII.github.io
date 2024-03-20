---
layout: post
toc: true
toc_sticky: true
title: "나만의 github blog 만들기(windows)"
excerpt: "github blog 생성부터 글 작성까지 해보자!"
date: 2024-03-20
writer: 이주영
categories: github blog
tags: [github blog, jekyll, github, git, markdown]
---
> 깃허브 블로그는 진입장벽이 높다고 들어서 엄두를 못냈었는데 이번 기회에 만들어보았다. 만드는 과정에서 일어난 여러 트러블 슈팅 또한 기록해두었다.

# 1. ruby 설치
![스크린샷 2024-03-20 114139](https://github.com/hobbyscripterII/about-play/assets/135996109/fb4defe5-a360-42ec-829d-7921111c240a)

[ruby download link](https://rubyinstaller.org/downloads/)

<br>

우측을 보면 '어떤 버전을 설치해야 할 지 모르겠으면 3.2x 버전을 다운로드하라'는 친절한 설명이 있다.

해당 버전을 다운로드한다.

(설치 과정은 버튼 누를 거 없이 그대로 쭉 설치해주면 된다)

cmd 창에서 `ruby -v` 명령어로 루비 버전을 확인했을 때 아래와 같이 출력되면 설치에 성공한 것이다.

```shell
$ ruby -v
ruby 3.2.3 (2024-01-18 revision 52bb2ac0a6) [x64-mingw-ucrt]
```

# 2. jekyll 설치
jekyll은 ruby가 설치되어 있다면 쉽게 설치가 가능하다.

아래와 같이 cmd 창에서 jekyll 설치 명령어를 입력해보자.

```shell
$ gem install jekyll
$ gem install bundle
```

jekyll 또한 -v 명령어로 버전 확인 시 아래와 같이 출력되면 설치에 성공한 것이다.

```shell
$ jekyll -v
jekyll 4.3.3
```

# 3. repository 생성
![스크린샷 2024-03-20 115008](https://github.com/hobbyscripterII/about-play/assets/135996109/dd8c9faa-42c8-4597-871f-6f835c9e43f2)
나는 이미 해당 레파지토리가 생성되어있기 때문에 해당 에러는 무시하고 보면된다.

레파지토리의 이름은 `{github_username}.github.io`로 지정한다.

# 4. repository clone
이제 내 컴퓨터에서 위에서 만든 레파지토리를 클론해야한다.

내가 원하는 디렉토리 내에서 터미널을 열거나 cmd창에서 `cd` 명령어로 클론받을 디렉토리에 이동하자.

그리고 클론받을 디렉토리에서 아래와 같이 명령어를 입력한다.

```shell
$ git clone https://github.com/{github_username}/{github_username}.github.io
```

# 5. jekyll 사이트 생성
4번에서 실행한 명령어를 통해 내가 원하는 디렉토리에 clone된 github repository를 확인할 수 있을 것이다.

이제 여기다가 jekyll 사이트를 생성해보자.

clone 받은 로컬 디렉토리에서 아래와 같이 명령어를 입력한다.

```shell
$ jekyll new ./
```

위의 명령어는 해당 디렉토리에 jekyll 사이트를 생성하는 명령어이다.

이제 jekyll bundle tool을 설치하고 update를 진행한다.

```shell
$ bundle install
$ bundle update
$ bundle install
```

이후 설치가 잘 되었는지 확인하기 위해 로컬에서 서버를 실행해본다.

```shell
$ bundle exec jekyll serve
```

위의 명령어를 통해 아래와 같이 출력되었다면 성공한 것이다.

```shell
Auto-regeneration: enabled for 'path:/{github_username}.github.io'
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.
```

그리고 server address에 적힌 `http://127.0.0.1:4000` url로 접속했을 때 `welcome to jekyll!`이라는 문구가 출력되었다면 jekyll 사이트가 생성된 것이다!

이제 아래부터는 내가 트러블슈팅한 흔적들이 꽤나 남아있는 jekyll 테마 적용부터 github action으로 build하는 과정이 적혀져있다.

같이 적용하면서 살펴보자!

# 6. jekyll 'Chirpy' 테마 다운로드
> 처음 jekyll 사이트를 생성하면 아무것도 없는 빈 사이트만 노출된다. 따라서 우리는 원하는 테마를 적용하기 위해 아래 링크에서 테마를 다운받도록 한다.
> 
> 나는 Chirpy 테마를 적용했으므로 Chirpy 테마 적용을 기준으로 작성한다.(다른 테마는 적용 방식이 조금씩 다른 것 같았다. 다른 테마를 적용하고 싶다면 다른 포스팅을 확인하는 것이 좋을 것이다)

http://jekyllthemes.org 에서 Chirpy 테마를 다운로드하고 압축을 푼다.

이후 클론받은 디렉토리에 다운로드 받은 파일들을 붙여넣는다. 중복되는 파일들이 몇개 있기에 덮어씌울거냐고 묻는데 덮어씌우면 된다.

그리고 아래에 파일들을 삭제 및 변경해준다.

- `Gemfile.lock` 파일 삭제
- `_posts` 디렉토리 삭제
- `docs` 디렉토리 삭제
- `path:\{github_username}.github.io\.github\workflows` 내에 `pages-deploy.yml.hook`을 제외한 모든 파일 삭제
- `pages-deploy.yml.hook`을 `pages-deploy.yml`으로 변경

이후 로컬 레파지토리에서 cmd창을 켜고 아래의 명령어를 실행해 테마를 적용시킨다.

```shell
$ bundle install
```

# 7. 블로그 설정 변경
이제 테마를 적용했으니 블로그를 설정해보자.
레파지토리의 루트 경로에 있는 `_config.yml`을 실행한다.

공통적으로 맞춰적어야 할 부분을 제외한 나머지 부분들은 개인 정보이므로 알아서 기입하면 된다.

**lang**: ko - 언어 선택(ko - 한국어) <br>
**timezone**: Asia/Seoul - 시간대 선택(Asia/Seoul - 한국) <br>

**title**: 블로그 타이틀(제목) <br>
**tagline**: 블로그 서브 타이틀 <br>
**description**: 블로그 설명 <br>
**url**: "https://{github_username}.github.io" <br>
**github.username**: {github_username} <br>
**theme_mode**: 블로그 테마 모드(다크 모드/라이트 모드) <br>
**avatar**: 프로필 사진(나는 github 프로필 url을 입력했다) <br>
**paginate**: 한 페이지에 나타낼 글 개수(default - 10)

외에 social 부분은 알아서 기입하면 된다.

모든 설정을 완료하고 아래 명령문을 통해 로컬 서버를 실행해본다.

```shell
$ bundle exec jekyll serve
```

![스크린샷 2024-03-20 125726](https://github.com/hobbyscripterII/about-play/assets/135996109/3eee8bc9-8c3a-4d19-8baa-37768c148966)

위와같이 화면에 나타나면 깃허브 블로그 만들기 2/3은 성공이다.

#### ⛔ 블로그 내에서 페이지 전환 시 계속 나타나는 'assert/js/dist/*.min.js Not Found'

화면에 jekyll 테마가 잘 적용되었다.

그러나 화면을 전환활 때마다 'assert/js/dist/*.min.js' 파일을 찾을 수 없다는 문구가 로그에 찍혀있었다.

해당 에러는 node.js 모듈을 설치하고 initial과 build를 하지 않으면 발생하는 에러로 해당 에러를 해결하지 않으면 블로그 기능이 정상적으로 작동하지 않는다는 것을 깨달았다.

해당 이슈를 해결하기 위해 node.js가 깔려있어야 하므로 [https://nodejs.org/en/download](https://nodejs.org/en/download)에서 다운로드 하자.

node.js를 다운받았으면 cmd에서 로컬 레파지토리 메인 루트로 이동 후 아래와 같이 npm을 설치 및 빌드 해주면 이슈가 해결된다.

```shell
$ npm install
$ npm run build
```

만약 `NODE_ENV은(는) 내부 또는 외부 명령, 실행할 수 있는 프로그램. 또는 배치 파일이 아닙니다.`라는 에러 발생시 아래 명령어를 실행 후 다시 빌드한다.

```shell
$ npm install -g win-node-env
$ npm run build
```
이후 형광색으로 `created assets/js/dist/*.min.js`가 로그에 여러 줄 뜨는 것을 확인했다면 해당 에러가 더 이상 발생하지 않을 것이다.

# 8. 테스트 포스팅 작성
로컬 레파지토리 메인 루트에 `_posts` 디렉토리를 생성한다.

```shell
├─.github
├─.husky
├─.jekyll-cache
├─.vscode
├─tools
├─_data
├─_includes
├─_javascript
├─_layouts
├─_plugins
├─_posts
├─_sass
├─_site
├─asserts
├─node_modules
└─_tabs
```

그럼 위와 같은 구조로 디렉토리가 생성될 것인데 해당 `_posts` 디렉토리에 `.md` 확장자를 가진 파일을 생성한다.

이 때, 제목은 `yyyy-MM-dd-제목.md` 형식이여야 한다.

(예시)
`2024-03-20-테스트.md`

```yml
---
title: 테스트
date: 2024-03-20
categories: [daily]
# tags: []
---
깃허브 블로그 테스트 블로깅
```

간단히 테스트용으로 작성한 것이므로 위와 같이 간단하게만 입력해준다.

**title** <br>
title은 쌍따옴표로 묶어도 상관없고 안묶어도 상관은 없다.

**date** <br>
date는 길게 입력할 필요는 없고 저 정도로 간단하게만 입력하면 된다.

**category** <br>
category는 대괄호를 생략하든 안하든 인자 값 받듯이 `categories: github blog` 와 같이 입력하게 되면 좌측의 '카테고리명1'은 메인 카테고리, 우측의 '카테고리명2'는 서브 카테고리로 지정되어 아래와 같이 출력된다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/dfced3e3-5f66-4033-88d1-4e559638e81f)

**tag** <br>
tag는 배열 형태로 입력받으며 `tags: [github blog, jekyll, github, git, markdown]`와 같이 입력했을 경우 아래와 같이 출력된다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/ddd048e6-2fe5-4c7d-87f0-e8437af61707)

`.md` 파일을 저장했다면 다시 로컬 레파지토리 메인 루트에서 아래 명령어를 통해 서버를 실행시켜 포스팅이 등록되었는지 확인한다.

```shell
$ bundle exec jekyll serve
```

#### ⛔ 메인 페이지 포스팅 미출력
category나 tags 페이지를 통해 포스팅이 정상적으로 등록되었다는 것을 확인했지만 메인 페이지에서 포스팅이 미출력되는 이슈가 있었다.

참고하면서 만든 블로그에서 미리 발생했던 에러라 간단하게만 적자면 `path:\{github_username}.github.io\_layouts`의 home.html에서 아래와 같이 소스코드를 수정해주면 된다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/ddf474fe-fda0-4293-80e6-5d70b19c4093)

for post in posts를 for post in site.posts로 변경한다.


# github action에 build 및 배포

![스크린샷 2024-03-20 134922](https://github.com/hobbyscripterII/about-play/assets/135996109/df6d5ac1-f6c8-4783-9b64-f2f03a75b9d4)

일단 github의 레파지토리에 settings - pages에서 build and deployment의 source를 `GitHub Actions`로 변경한다.

이후 로컬 레파지토리 메인 루트에서 아래 명령어를 실행한다.

```shell
$ git add -A
$ git status
$ git commit -m "Create github blog"
$ git push
```

커밋 메세지는 원하는대로 적어주면 된다.

cmd 창에서 push를 진행할 경우 아래와 같이 빌드 과정을 확인할 수 있다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/b5278ed9-b283-4700-99e3-1794c2b8cbff)

나는 첫번째 빌드에서 특정 폴더가 생략되었기 때문에 위와 같이 에러가 발생했는데 무사히 빌드가 완료되었다면 `https://{github_username}.github.io/`으로 접속했을 경우 jekyll 테마가 적용된 것은 물론 등록한 포스팅 또한 확인할 수 있다.

끝!

#### ⛔ Process completed with exit code.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/42d97de0-f48c-4cab-83d4-dae7cdf4f6b7)

```shell
Run bundle exec htmlproofer _site \
Running 3 checks (Scripts, Links, Images) in ["_site"] on *.html files...

Checking 14 internal links
Checking internal link hashes in 0 files
Ran on 11 files!

For the Scripts check, the following failures were found:

* At _site/404.html:1:

  internal script reference /assets/js/dist/commons.min.js does not exist

* At _site/about/index.html:1:

  internal script reference /assets/js/dist/page.min.js does not exist

* At _site/archives/index.html:1:

  internal script reference /assets/js/dist/misc.min.js does not exist

* At _site/categories/daily/index.html:1:

  internal script reference /assets/js/dist/misc.min.js does not exist

* At _site/categories/index.html:1:

  internal script reference /assets/js/dist/categories.min.js does not exist

* At _site/index.html:1:

  internal script reference /assets/js/dist/home.min.js does not exist

* At _site/posts/테스트/index.html:1:

  internal script reference /assets/js/dist/post.min.js does not exist

* At _site/tags/index.html:1:

  internal script reference /assets/js/dist/commons.min.js does not exist


HTML-Proofer found 8 failures!
Error: Process completed with exit code 1.
```

github action은 처음 사용해봐서 살짝 어리둥절 했었는데 읽어보니 공통적으로 `/assets/js/dist/**`에 해당되는 모든 파일들을 찾을 수 없다는 것이였다.

`git add -A` 명령어를 사용했으므로 기본적으로 메인 루트에 있던 모든 파일들이 다 추가됐을텐데 누락된 디렉토리가 있나보다 싶어 확인하니 `/assets` 디렉토리가 push가 되지 않았다.

![image](https://github.com/hobbyscripterII/about-play/assets/135996109/654520f4-66bd-488d-b544-c6c03f25bd96)

위와 같이 `Add file - Upload files`를 클릭해 누락된 디렉토리를 넣고 push 해주니 무사히 build가 되는 것을 확인할 수 있었다!

---
**참고 레퍼런스** <br>
[github 블로그 만들기](https://devpro.kr/posts/Github-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0-(1)/) <br>
[git action 자동 배포 에러 해결기](https://seobie.github.io/blog/git-action-struggles) <br>
[github jekyll 테마 적용하다 만난 오류들(Chirpy 테마)](https://velog.io/@lzlko/github-%EB%B8%94%EB%A1%9C%EA%B7%B8) <br>
[github pages 블로그 만들기 - 테마 적용하기(Chirpy)](https://ree31206.tistory.com/entry/github-pages-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EB%A7%8C%EB%93%A4%EA%B8%B0-%ED%85%8C%EB%A7%88-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0Chirpy)