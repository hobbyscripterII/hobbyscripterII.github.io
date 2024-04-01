---
title: "IntelliJ IDEA로 build 시 관련 이슈들"
date: 2024-03-30 22:54:00 +09:00
writer: 이주영
categories: spring
tags: [spring, issue, '@PathVariable', JPA, JPQL]
toc: true
toc_sticky: true
---
![image](https://github.com/hobbyscripterII/about-play/assets/135996109/ecdf976c-acb8-497d-9510-979727e5501b){: width="50%" .normal}

build 환경이 다를 경우 발생할 수 있는 이슈를 2개 발견하고 작성하는 IntelliJ IDEA로 build 시 관련 이슈 모음 포스팅이다!

## 1. Name for argument of type [long] not specified 이슈

```
Name for argument of type [long] not specified, and parameter name information not available via reflection.
Ensure that the compiler uses the '-parameters' flag.
```

원룸에선 잘만 되던 기능이(게시글 출력) 본가와서 작업하려고 pull을 받으니 갑자기 안됐다.

```java
    @GetMapping("/read-playlist/{iboard}")
    public String selPlaylistBoard(@RequestParam(name = "code") int code, @PathVariable long iboard, Model model) {
        getTitle(code, model);
        model.addAttribute("board", boardService.selPlaylistBoard(iboard));
        return "/board/read-playlist";
    }
```

위의 코드에서 에러가 터졌는데 `@PathVariable`에 name을 생략해서 발생한거고 조금 더 찾아보니 빌드 시 잡아놓은 환경이 서로 달라서였다.

아래를 확인해보자.

![화면 캡처 2024-03-30 224734](https://github.com/hobbyscripterII/about-play/assets/135996109/291d2d9c-8f67-4283-b495-e5e8bb10359c)

내 본가 컴퓨터에선 빌드 시 IntelliJ IDEA가 빌드하도록 되어 있었는데 아마 원룸에선 gradle로 빌드하게 끔 되어 있었던 것 같다.(소스코드 건드리기 전에 발생한 에러라 다른 점은 딱히 못찾겠다. 만약에 원룸가서 다시 찾는다면 수정하겠다!)

이 쯤 되면 **빌드할 때 gradle로 잡는거랑 IntelliJ IDEA로 잡는거랑 뭐가 다른가** 싶을 것이다.

일단 스프링 부트 3.2 버전 이전까지는 바이트 코드를 파싱해서 매개변수 이름을 추론하려고 시도했으나 3.2 버전부터는 이런 시도를 하지 않는다고 한다.

그러나 gradle 같은 경우에는 컴파일 시점에 -parameters 옵션을 자동으로 적용해주기 때문에 `@RequestParam`과 `@PathVariable` 어노테이션 사용 시 name 속성으로 이름을 지정해주지 않아도 자동으로 컴파일러 시점에 파라미터 이름 인식이 가능하다.

그러나 IntelliJ iDEA로 적용할 경우에는 -parameter 옵션을 자동으로 적용하지 않기 때문에 설정에서 수동으로 -parameters 옵션을 적용하거나 아래와 같이 `@RequestParam` 혹은 `@PathVariable` 어노테이션 사용 시 name을 명시해주어야 한다.

```java
    @GetMapping("/read-playlist/{iboard}")
    public String selPlaylistBoard(@RequestParam(name = "code") int code, @PathVariable(name = "iboard") long iboard, Model model) {
        getTitle(code, model);
        model.addAttribute("board", boardService.selPlaylistBoard(iboard));
        return "/board/read-playlist";
    }
```

## 2. JPQL 사용 시 For queries with named parameters you need to provide names for method parameters 이슈

```
For queries with named parameters you need to provide names for method parameters;
Use @Param for query method parameters, or when on Java 8+ use the javac flag -parameters
```

위에서 발생한 'Name for argument of type [long] not specified 이슈'와 비슷한 맥락이다.

```java
public interface PlaylistRepository extends JpaRepository<PlaylistEntity, Long> {
    @Modifying
    @Query("delete from PlaylistEntity p where p.boardEntity.iboard = :iboard")
    void deleteByIboard(long iboard);
}
```

게시글 삭제 시 위의 repository의 deleteByIboard를 실행하는데 이 때 발생하는 이슈로 파라미터에 `@Param`을 사용하여 name을 직접 정의해주거나 설정의 `-parameters`를 사용하라고 한다.

> `@Param` mybatis, JPQL 등의 쿼리문에 변수를 바인딩해주는 어노테이션

```java
public interface PlaylistRepository extends JpaRepository<PlaylistEntity, Long> {
    @Modifying
    @Query("delete from PlaylistEntity p where p.boardEntity.iboard = :iboard")
    void deleteByIboard(@Param("iboard") long iboard);
}
```

위와 같이 직접 명시해주거나 설정에서 `-parameters`를 추가해주면 된다.

---
**참고 레퍼런스** <br>
[Build Gradle VS IntelliJ IDEA](https://pamyferret.tistory.com/62) <br>
[@PathVariable name 생략 질문드립니다.](https://www.inflearn.com/questions/1087879/pathvariable-name-%EC%83%9D%EB%9E%B5-%EC%A7%88%EB%AC%B8-%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4) <br>
[Name for argument of type, @PathVariable name 생략 에러](https://olrlobt.tistory.com/75)