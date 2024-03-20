---
title: "github profile에 blog post 자동 업데이트하기"
excerpt: "github profile에 blog post를 자동으로 업데이트 해보자!"
date: 2024-03-20 22:33:00 +09:00
writer: 이주영
categories: github profile
tags: [github blog, profile, github]
toc: true
# toc_sticky: true
---
github blog는 나중에 하면서 조금씩 더 꾸미기로하고 이번에는 github profile에 블로그 포스팅 목록을 내림차순으로 보여주는 방법이 있다고 해서 적용해봤다!

![image](https://i.pinimg.com/564x/48/85/6d/48856d1e68fd5daa19315451f571dd2c.jpg){: width="50%"}

go go..

<br>
<br>
<br>
<br>

# 1. github profile `README.md` 수정
일단 먼저 github profile의 README.md를 수정해줘야 한다.

```md
<!-- BLOG-POST-LIST:START -->
<!-- BLOG-POST-LIST:END -->
```

틀리면 안되니까 복붙하자.

# 2. workflow yml 생성
github profile 레파지토리에 들어간다.

'Add file - Create new file'을 누른다.

![image](https://github.com/hobbyscripterII/csharp/assets/135996109/d1d3aa17-7193-43fb-b9fc-d11444288a05)

`.github/workflows/` 경로로 `blog-post-workflow.yml` 파일을 생성한다.

```yml
name: Latest blog post workflow
on:
  schedule: # Run workflow automatically
    - cron: '0 * * * *' # Runs every hour, on the hour
  workflow_dispatch: # Run workflow manually (without waiting for the cron to be called), through the GitHub Actions Workflow page directly
permissions:
  contents: write # To write the generated contents to the readme

jobs:
  update-readme-with-blog:
    name: Update this repo's README with latest blog posts
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Pull in dev.to posts
        uses: gautamkrishnar/blog-post-workflow@v1
        with:
          feed_list: "https://{github_username}.github.io/feed"
```

제일 밑에있는 `feed_list`가 중요한데 해당 url은 아래와 같이 RSS 아이콘을 클릭했을 때 상단 url에 '.xml'을 제외한 나머지 url을 입력하면 된다.

![스크린샷 2024-03-20 230942](https://github.com/hobbyscripterII/csharp/assets/135996109/461b2f52-f90b-48a3-9c33-ca82e1fcc01c)

# 3. run workflow

![스크린샷 2024-03-20 224948](https://github.com/hobbyscripterII/csharp/assets/135996109/25731317-0e6f-4b8f-942c-1238ba7637ac)

마지막이다!

actions - Latest blog post workflow - Run workflow - Run workflow를 차례대로 눌러주면 끝이다.

![image](https://github.com/hobbyscripterII/hobbyscripterII/assets/135996109/cc4f5951-048c-40a5-8959-86563d0040b9)

적용된 모습을 확인할 수 있다!

---
**참고 레퍼런스** <br>
[깃허브 프로필에 블로그 포스트 자동 업데이트 하는 법](https://velog.io/@hameo/%EA%B9%83%ED%97%88%EB%B8%8C-%ED%94%84%EB%A1%9C%ED%95%84%EC%97%90-%EB%B2%A8%EB%A1%9C%EA%B7%B8-%ED%8F%AC%EC%8A%A4%ED%8A%B8-%EC%9E%90%EB%8F%99-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8%ED%95%98%EB%8A%94%EB%B2%95) <br>
[blog-post-workflow](https://github.com/gautamkrishnar/blog-post-workflow)