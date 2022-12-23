---
layout: default
title: Spring Data Common 4
parent: 스프링데이터JPA_백기선
nav_order: 8
---

- Spring Data Common Web 기능
  - DomainClassConverter
  - HandlerMethodArgumentResolver  
    

## Spring Data Common Web 기능
Spring Data Common 이 제공해주는 Web support 기능들에 대해서 알아본다.

### Domain Class Converter
```java
public class DomainClassConverter<T extends ConversionService & ConverterRegistry>
		implements ConditionalGenericConverter, ApplicationContextAware
```
가 Converter Registry 에 들어가서 핸들러의 파라미터를 컨버팅 하는 용도로 사용된다.
내부적으로 DomainClassConverter내에 선언된 아래 두 컨버터가 사용된다.
컨버팅 과정에서 Repository 를 사용해 조회한다.

```java
	private Optional<ToEntityConverter> toEntityConverter = Optional.empty();
	private Optional<ToIdConverter> toIdConverter = Optional.empty();
```

그래서 아래와 같이 핸들러에서 바로 entity 조회가 가능하다.
```java
    @GetMapping("/posts/{id}")
    public String getAPost(@PathVariable Long id) {
        Optional<Post> byId = postRepository.findById(id);
        Post post = byId.get();
        return post.getTitle();
    }
```
```java
    @GetMapping("/posts/{id}")
    public String getAPost(@PathVariable(“id”) Post post) {
        return post.getTitle();
    }
```

### HandlerMethodArgumentResolver
핸들러에서 Pageable 을 구성하는 아래 파라미터가 들어온다는 전제하에 바로 Pageable 객체를 바로 받을 수 있다.
- page
- size
- sort(ex. sort=createdAt,desc)

```java
    @GetMapping("/posts")
    public void testHandler(Pageable pageable){
        
    }
```
