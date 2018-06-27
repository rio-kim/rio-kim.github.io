---
layout: post
title: Make easy java collection code
categories: Java
tags: java collection
---

자바 코드를 작성하다보면 간단한 자료구조들을 만들어 사용하는 경우들이 있다. List, Set, Map 등이 그런 간단한 자료구조들인데, 보통 new로 만들어서 사용하곤 한다. 테스트코드를 작성할 때도 역시 큰 의미없이 형태를 구성하는 list나 map등을 만드는 경우가 많이 생긴다. 이런 경우에 **코드량을 줄이고 가독성을 높이기 위해서** 일반적으로 new로 생성하는 list 또는 map 형태를 피하고 아래와 같은 형태를 사용하곤 했다.

##### 1. List 생성시

자바 기본 라이브러리 중 Arrays 클래스의 asList를 사용해서 코드의 가독성을 높인다.

~~~java
List<String> fruits = Arrays.asList("apple", "banana", "coconut");
~~~

static import를 활용할 수 있다면 아래와 같이 쓸 수 있다.

~~~java
import static java.util.Arrays.*;

List<String> fruits = asList("apple", "banana", "coconut");
~~~

위 코드는 아래의 코드와 완전히 같은 역할을 한다.

~~~java
List<String> fruits = new ArrayList<>();
fruits.add("apple");
fruits.add("banana");
fruits.add("coconut");
~~~

List 내부의 값들을 명시적으로 보여주며 코드량을 획기적으로 줄여주어 굉장히 많이 사용했다.

##### 2. 단 하나의 객체만 들어간 List 생성시

~~~java
List<String> fruits = Collections.singletonList("apple");
~~~

static import를 활용할 수 있다면 아래와 같이 쓸 수 있다.

~~~java
import static java.util.Collections.singletonList;

List<String> fruits = singletonList("apple");
~~~

그냥 asList 내에 파라미터 하나만 넘기는 방법으로 사용해도 결과는 같다.

##### 3. ImmutableList 활용

google guava 라이브러리를 활용한 경우 ImmutableList를 활용하여 작성하기도 한다. 특히 테스트 데이터를 만드는 경우 중간에 값 변경을 하는 경우가 거의 없기 때문에 사용할 수 있다. 하지만 asList라는 함수를 활용하는 빈도가 높아 거의 쓰지는 않았다.

~~~java
List<String> fruits = ImmutableList.of("apple", "banana");
~~~

##### 4. Map 생성시

google guava 라이브러리 내의 ImmutableMap을 자주 사용했다. 특히 api 테스트시 간단하게 request body를 만들어 보낼 때 활용하면 좋다. 객체를 따로 만들지 않아도 **IDE 툴의 code indent만 잘 맞추어 작성하면 key-value쌍이 가독성 있게 읽혀서 많이 사용하곤 했다.**

~~~java
Map<String, Object> requestMap = ImmutableMap.of(
                "name", "rio",
                "age", 33,
                "gender", "male"
        );
~~~

아래에 new HashMap으로 생성하는 코드를 위의 코드로 대체해서 쓸 수 있다.

~~~java
Map<String, Object> requestMap = new HashMap<>();
requestMap.put("name", "rio");
requestMap.put("age", 33);
requestMap.put("gender", "male");
~~~

key-value쌍이 단 하나인 Map을 만들어야 하는 경우 자바 기본 라이브러리 중 Collections 클래스의 singletonMap을 static import를 활용해 사용하는 일이 많았다.

~~~java
import static java.util.Collections.singletonMap;

Map<String, Object> singletonMap = singletonMap("key", "value");
~~~

##### 5. 비어 있는 Collection 생성시

역시 자바 기본 라이브러리 중 Collections 클래스 내에서 제공되는 empty~ 메서드를 이용하곤 했다.
~~~java
import static java.util.Collections.*;

List<String> emptyList = emptyList();
Map<String, Object> emptyMap = emptyMap();
Set<String> emptySet = emptySet();
~~~

위에 소개한 기법 외에도 간단한 자료구조를 만들어내는 방법들은 많을 것이다. 다만 프로젝트에서 자동화 테스트 지원을 하면서 여러가지 방법 중에 가독성 측면에서 이득을 보고 습관적으로 잘 사용했던 몇 가지 메서드만 잊어버리지 않기 위해 기록해 둔다.

보통은 생성한 Map, List 등을 특정 변수로 받아서 활용하기보다는 파라미터로 바로 **inline으로 전달하는 목적으로** 많이 쓰이기 때문에 실제 프로젝트 코드에 적용하는 경우 예시로 든 것보다 가독성 측면에서 더 좋은 효과를 볼 수 있다.
