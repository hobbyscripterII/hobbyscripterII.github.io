---
title: "JPQL 사용 시 For queries with named parameters you need to provide names for method parameters 이슈"
date: 2024-03-31 11:00:00 +09:00
writer: 이주영
categories: spring
tags: [spring, JPA, JPQL, issue]
toc: true
toc_sticky: true
---
```
For queries with named parameters you need to provide names for method parameters;
Use @Param for query method parameters, or when on Java 8+ use the javac flag -parameters
```

어제 발생했던 [Name for argument of type [long] not specified 이슈](https://hobbyscripterii.github.io/posts/Name-for-argument-of-type-long-not-specified-%EC%9D%B4%EC%8A%88/)와 비슷한 맥락이다.

<br>

```java
public interface PlaylistRepository extends JpaRepository<PlaylistEntity, Long> {
    @Modifying
    @Query("delete from PlaylistEntity p where p.boardEntity.iboard = :iboard")
    void deleteByIboard(long iboard);
}
```

게시글 삭제 시 위의 repository의 deleteByIboard를 실행하는데 이 때 발생하는 이슈로 파라미터에 `@Param`을 사용하여 name을 직접 정의해주거나 설정의 `-parameters`를 사용하라고 한다.

> `@Param` mybatis, JPQL 등의 쿼리문에 변수를 바인딩해주는 어노테이션

<br>

```java
public interface PlaylistRepository extends JpaRepository<PlaylistEntity, Long> {
    @Modifying
    @Query("delete from PlaylistEntity p where p.boardEntity.iboard = :iboard")
    void deleteByIboard(@Param("iboard") long iboard);
}
```

위와 같이 직접 명시해주거나 설정에서 `-parameters`를 추가해주면 된다. 아마 이것도 어제 발생한 이슈처럼 빌드 시 IntelliJ IDEA로 돌리는 환경에서만 터지는 이슈인 것 같다.