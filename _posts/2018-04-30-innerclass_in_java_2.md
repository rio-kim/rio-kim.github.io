---
layout: post
title: InnerClass in Java project 2
categories: Java
tags: java class innerclass
---

InnerClass를 조금 더 간단하게 사용하는 방법

~~~java
public class Customer {

    private String name;
    private int age;
    private String address;
    private Dog dog;

    public static class Dog {
        private String name;
        private int age;
    }
}
~~~

위와 같이 내부 클래스 자체를 **public static** 으로 작성한다.

~~~java
Customer.Dog dog = new Customer.Dog();
~~~

객체 생성은 이와 같이 선언한다. 원래의 코드와 다른게 없어 보이지만 사용하는 측에서 좀더 코드가 깔끔해진다.

여기서 InnerClass 자체를 **static import** 문을 사용해서 쓰면 아래와 같이 쓸 수 있다. 실제로는 내부 클래스이지만 코드를 읽을 때는 일반 객체를 만든 것과 비슷하게 읽혀서 가독성이 좋다. 테스트 코드를 작성하는 입장에서만 볼 때는, 도메인 단위로 내부 클래스를 만들고 난 후 자연스럽게 아래와 같이 많이 쓰게 된다.

~~~java
import rio.kim.sample.Customer.Dog;

Dog dog = new Dog();
~~~
