---
layout: post
title: Response parsing to class model in Rest-Assured
categories: Rest-Assured
tags: rest-assured response parsing
---

Rest-Assured 라이브러리를 사용해서 받은 response를 모델 객체로 파싱해서 쓸 때 아래와 같이 간단하게 쓸 수 있다.

~~~java
MyClass myClass = RestAssured.given()
        .when().get("/blahblah")
        .then().statusCode(200)
        .extract().as(MyClass.class);

MyClass myClass = RestAssured.given()
        .when().get("/blahblah")
        .as(MyClass.class);
~~~

then()~extract() 구문은 빠져도 상관없다. then이후 특정 검증이 필요한 경우에 저런 식으로 사용하곤 했다. MyClass는 response를 분석해서 만들면 되고 ObjectMapper만 잘 설정해 두면 response를 굉장히 편하게 관리할 수 있다. 그 부분은 나중에 다시 정리.
