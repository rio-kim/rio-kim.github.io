---
layout: post
title: Custom assertions in AssertJ
categories: Testing
tags: test assertj
---

자동화 테스트 지원을 하며 AssertJ 라이브러리의 custom assertion을 만들어서 사용하였는데 사용하면서 느낀 점을 정리해본다. 작성 방법은 공식 홈페이지에 워낙 쉽게 설명되어 있기 때문에([Creating assertions specific to your classes](http://joel-costigliola.github.io/assertj/assertj-core-custom-assertions.html)) 이 포스트에서는 따로 설명하지 않는다.

##### 코드 가독성이 좋아진다.

내가 느낀 제일 가장 큰 장점은, custom assertion을 만들어서 사용한 경우 크게 두 가지 케이스에 대해서 코드 가독성이 눈에 띄게 좋아졌다는 점이다.

우선, **객체 내부의 여러 필드에 대해 값을 검증하는 경우** 가독성이 좋아지는 케이스이다. 아래의 코드를 보면 확실하게 느껴진다. Customer라는 객체 내부에 여러 필드를 검증하는 예시를 간단하게 요약해 표현했다.

~~~java
public class Customer {

    private int customerId;
    private String name;
    private int age;
    private String email;
    private String phoneNumber;
    private List<Discount> discountList;

    public static class Discount {
        private int discountId;
        private String name;
    }
}
~~~

이 경우 전체 필드의 값에 대한 검증을 할 수 있는데 일반적인 AssertJ 라이브러리를 활용하면 아래처럼 쓸 수 있다.

~~~java
@Test
public void customerTest() {

    Customer rio = getCustomerById(231);

    assertThat(rio.getCustomerId()).isEqualTo(231);
    assertThat(rio.getName()).isEqualTo("rio");
    assertThat(rio.getAge()).isEqualTo(33);
    assertThat(rio.getGender()).isEqualTo(Gender.male);
    assertThat(rio.getAddress()).isEqualTo("서울");
    assertThat(rio.getEmail()).isEqualTo("rio.dykim@gmail.com");
    assertThat(rio.getPhoneNumber()).isEqualTo("010-1111-2222");
    assertThat(rio.getDiscountList()).hasSize(2);
    List<String> discountNameList = rio.getDiscountList().stream().map(Customer.Discount::getDiscountName).collect(toList());
    assertThat(discountNameList).contains("연회비할인", "가족회원할인");
}
~~~

Custom assertion을 활용하는 경우 위 코드를 아래처럼 표현할 수 있다.

~~~java
@Test
public void customerTest() {

    Customer rio = getCustomerById(231);

    CustomerAssert.assertThat(rio)
            .customerId(231)
            .name("rio")
            .age(33)
            .gender(Gender.male)
            .address("서울")
            .email("rio.dykim@gmail.com")
            .phoneNumber("010-1111-2222")
            .hasDiscountCount(2)
            .containsDiscount("연회비할인", "가족회원할인");

}
~~~

**어떤 필드가 어떤 값을 가지는지 검증하고자 하는 의도가 코드에 자연스럽게 드러난다.** 위의 코드와 완전히 같은 일을 하지만 가독성 측면에서 상당한 효과를 볼 수 있다.

그리고 다른 케이스에서 장점이라 볼 수 있는 점은 **검증문 자체에 비즈니스 로직이 필요한 경우이다.** 위 케이스의 예를 들면 필드에서는 age와 gender, address만 가지고 있고 서울 경기권에 사는 30대 남자인지 아닌지를 판단하는 비즈니스 로직이 중요한 부분이라면 테스트 코드에서는 그 부분을 따로 아래와 같이 검증해야 한다.

~~~java
Customer rio = getCustomerById(231);
assertThat(rio.getAge()).isBetween(30, 39);
assertThat(rio.getGender()).isEqualTo(Gender.male);
assertThat(rio.getAddress()).isIn("서울", "경기");
~~~

또는 해당하는 조건들을 모아 변수로 뽑던지, private 메서드를 활용하던지 해서 true/false 형태의 검증도 가능하다.

~~~java
boolean is30sManInMetropolitanArea = rio.getAge() >= 30 && rio.getAge() < 40
        && rio.getGender().equals(Gender.male)
        && (rio.getAddress().equals("서울") || rio.getAddress().equals("경기"));
assertThat(is30sManInMetropolitanArea).isTrue();
~~~

이런 경우 custom assertion을 활용하여 간단히 아래처럼 표현할수 있다. **메서드 체이닝을 이용해 다른 검증과 함께 할 수 있는 장점도 살릴 수 있다.** 물론 내부 구현은 작성해야 하겠지만 테스트 코드를 읽어가는 흐름에서는 그 검증 로직이 코드에 노출되느냐 아니냐에 이슈를 두는 것이 아니기 때문에 큰 문제가 되지 않는다.

~~~java
@Test
public void customerTest() {

    Customer rio = getCustomerById(231);

    CustomerAssert.assertThat(rio)
            .customerId(231)
            .is30sManInMetropolitanArea();
}
~~~

대부분의 프로젝트가 위의 예시보다는 더욱 복잡한 구조의 모델을 가질 테고, 케이스도 훨씬 다양할 것이다. 프로젝트의 규모가 커지고 기간이 길어질수록 코드 가독성은 곧 생산성과 직결되는 문제이다. 실제 지원했던 프로젝트의 보안 규정 때문에 복잡한 코드들을 포스트에 공개할 수는 없지만, 수많은 테스트코드를 작성하며 다른 사람이 짠 테스트 코드도 봐야 하는 개발자 입장에서는 이런 부분들이 굉장히 큰 편의로 다가오게 된다.
