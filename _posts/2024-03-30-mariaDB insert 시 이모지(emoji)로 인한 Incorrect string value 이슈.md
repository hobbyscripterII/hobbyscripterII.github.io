---
title: "mariaDB insert 시 이모지(emoji)로 인한 Incorrect string value 이슈"
date: 2024-03-30 14:20:00 +09:00
writer: 이주영
categories: sql
tags: [sql, mysql, mariadb]
toc: true
toc_sticky: true
---
## 1. 이슈 원인

![스크린샷 2024-03-30 135428](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/3ae2c9f0-643e-4e9a-a589-89d377f63761)

위와 같이 테이블에 이모지를 넣으려고 할 때 `Incorrect string value: '\xF0\x9F\x98\x8A'`와 같은 에러가 발생했다.

이유는 기존 mariaDB의 문자셋(나같은 경우는 utf8mb3_bin였다)의 경우 최대 3bytes까지 지원하나 이모지(emoji)와 같은 특수문자들은 최대 4bytes가 필요하므로 테이블에 insert 시 잘못된 문자열 값이라는 에러를 발생시킨 것이다.

> 모든 utf8 기반의 인코딩 방식은 가변 길이 인코딩 방식을 지원한다. <br>
> utf8은 가변 3bytes를 사용하는 데 반해 utf8mb4는 내부적으로 한 문자를 표현할 때 최대 4bytes까지 사용한다.

## 2. 이슈 해결

![스크린샷 2024-03-30 141226](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/dee93418-4785-4d4e-b56c-7a38edd3ea7c)

테이블의 옵션에서 기본 조합을 `utf8mb4_general_ci`로 변경한다.

## 3. 결과

![스크린샷 2024-03-30 142415](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/bae26963-fd5c-465b-8cdf-f054ddede783)


![스크린샷 2024-03-30 141018](https://github.com/hobbyscripterII/to-do-list.io/assets/135996109/f6dbba0a-ddab-44ea-8a93-2ded85c25fc9)

테이블에도 잘 들어가고 화면에도 정상적으로 출력되는 것을 확인할 수 있다!

---
**참고 레퍼런스** <br>
[utf8 vs utf8mb4 차이는?](https://cirius.tistory.com/1769)