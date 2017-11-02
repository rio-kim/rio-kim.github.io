---
layout: post
title: InnerClass를 활용해보자 in Java project
categories: Java
tags: rest api library java
---

모델객체를 만들 때 여러 클래스가 중첩되게끔 구성하는 일이 많은데 하위에 있는 클래스가 여러개일 때 그 클래스들을 다 만들어주고나서 클래스 구성을 짜줘야 한다.

예를 들어보자. Customer라는 클래스를 만드는데 그 안에는 Customer의 정보를 담고 있는 필드들이 있고 그 필드 중에는 객체로 관리하게되는 필드도 있을 것이다. 내가 필요한 고객의 정보는 이름, 나이, 주소, **그리고 키우는 강아지의 이름과 나이다.** 그러면 Customer의 필드로 개이름, 개나이 이렇게 넣는 것보다 **Dog라는 객체를 만들어 그 Dog 안에 개의 이름과 나이를 넣는게 낫다.**

이렇게 구조를 짜면 클래스가 두개 생긴다. Customer와 Dog 클래스. **근데 Dog 클래스는 단독으로 쓰이지 않고 Customer안에서만 필요하다고 생각해보자.** (쓰다보니 말도안되는 예시를 드는거 같은데 프로젝트에서는 이런일이 빈번하다. 사내보안규정 때문에 그 코드를 가져올 수 없어서 혼자 생각한게 고작 이런 것들 ㅠㅠ) 그러면 불필요하게 Dog 클래스를 외부에 노출시킬 필요가 없다. 단지 Customer 안에서만 쓰이니까. 그러면 이렇게 만들 수 있다.

~~~java
public class ParentClass {

    private String name;
    private int age;
    private String address;
    private Dog dog;

    public class Dog {
        private String name;
        private int age;
    }
}
~~~

물론 Constructor와 Getter, Setter, toString 등의 기본적인 메서드는 알아서 만들어 주자. 코드가 쓸데없이 길어지는 걸 피하기 위해 여기서는 제외. 그럼 외부에서는 이걸 어떻게 쓰냐.

우선 innerClass의 선언은

~~~java
outerClass.InnerClass innerClass = outerClass.new InnerClass();
~~~

이와 같이 선언한다. 그러면 숨어있는 innerClass를 다른 클래스에서 쓸 수 있다.
아래의 코드를 참고하자.

~~~java
import org.junit.Test;

import static org.hamcrest.core.Is.is;
import static org.junit.Assert.*;

public class InnerClassTest {

    @Test
    public void innerClassTest() throws Exception {

        Customer customer = new Customer();
        customer.setName("Rio");
        customer.setAge(32);
        customer.setAddress("서울시 송파구");

        Customer.Dog dog = customer.new Dog(); // innerClass 선언부
        dog.setName("부침개");
        dog.setAge(2);
        customer.setDog(dog);

        assertThat(customer.getDog().getAge(), is(2)); // 성공

    }
}
~~~

또한 모델 객체가 아닌 경우에, 다시말해 innerClass에 메서드가 있는 경우에도 위에처럼 innerClass에 접근하여 메서드를 호출할 수 있다.

InnerClass인 Dog 클래스 내에 아래와 같은 메서드를 추가한다.
~~~java
public void printDogInfo() {
    System.out.println("나는 " + name + " 입니다. 내 나이는 " + age + "입니다.");
}
~~~

호출부는 다음과 같다. 아까 만든 코드 마지막 부분에 넣으면 된다.
~~~java
customer.getDog().printDogInfo();
~~~

이러면 정상적으로 출력

~~~
나는 부침개 입니다. 내 나이는 2입니다.
~~~
