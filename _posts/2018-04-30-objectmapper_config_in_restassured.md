---
layout: post
title: ObjectMapper configuration in Rest-Assured
categories: Rest-Assured
tags: rest-assured response parsing
---

Rest-Assured 라이브러리를 사용해서 받은 response를 모델 객체로 파싱해서 쓰는 방법에 대해서 간단하게 남겼는데(["Response parsing to class model in Rest-Assured"](https://rio-kim.github.io/testing/2018/04/30/restassured_response_to_class)) 내부적인 동작은 Rest-Assured가 가지고 있는 ObjectMapper가 response로 받은 json구조를 분석하여 Class에 자동으로 파싱해주는 것이다.

이때 어떤 api는 수많은 정보를 response에 전달하고, 사용하는 측에서는 response의 모든 값이 아니라 필요한 값만 클래스에 담고 싶은 경우가 있다.

이 경우에는 아래와 같이 ObjectMapper를 만들어 RestAssured에 설정해줄 수 있다.


~~~java
ObjectMapper objectMapper = new ObjectMapper()
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

RestAssuredConfig restAssuredConfig = new RestAssuredConfig()
        .objectMapperConfig(new ObjectMapperConfig().jackson2ObjectMapperFactory((cls, charset) -> objectMapper));

given().config(restAssuredConfig)
        .get("/blahblah")
        .then().statusCode(200)
        .body("name", is("rio"));
~~~

원하는 설정을 담은 custom한 objectMapper를 만든 후 RestAssuredConfig에 세팅한다. 그 이후 실제로 call하는 rest-assured 코드에서는 **.config()** 메서드에 미리 만들어둔 RestAssuredConfig를 담아서 호출한다.

참고로 저 위의 **DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES** 설정은 json을 class로 파싱하면서 모르는 프로퍼티가 나와도 무시하라는 뜻이다. 이 경우 response에 많은 정보가 담겨나와도 내가 원하는 class에 파싱하면서 class에 필요한 정보만 찾아 넣어주기 때문에 클래스 구조를 심플하게 가져갈 수 있다.

또, **MapperFeature.USE_ANNOTATIONS** 피쳐도 테스트 진행시 많이 사용했던 설정중 하나이다. 이는 실제 제품코드에서 사용하는 모델 객체에 필요해서 달아둔 어노테이션 설정을 다 무시하는 세팅이다.

objectMapper 설정을 통해 피쳐들을 disable하거나 enable할 수 있으므로 검색해보고 판단해서 쓰면 좋은 효과를 볼 수 있다.

다음과 같이 여러 피쳐들을 한번에 disable할 수도 있다.

~~~java
ObjectMapper objectMapper = new ObjectMapper()
        .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .disable(MapperFeature.USE_ANNOTATIONS);
~~~

다음과 같이 **호출되는 Rest-Assured 전역 설정으로** 사용할 수도 있다.

~~~java
RestAssured.config = restAssuredConfig;
~~~


##### 참고) Jackson2ObjectMapperBuilder

스프링 기반 프로젝트의 테스트에서는 아래와 같이 Jackson2ObjectMapperBuilder를 사용할 수 있다. 이유는 잘 모르겠지만 Jackson2ObjectMapperBuilder 클래스는 **org.springframework:spring-web** 라이브러리 안에 속해있다. 이름만 봐서는 당연히 jackson-databind 라이브러리에도 있을 것 같지만 없다. 특별한 경우가 아니면 쓸 일이 없지만 진행했던 프로젝트에서는 아래와 같이 특정 클래스의 serializer 또는 deserializer를 전역 설정으로 쓸 수 있어서 굉장히 편하게 사용했다.

~~~java
RestAssuredConfig restAssuredConfig = new RestAssuredConfig().objectMapperConfig(new ObjectMapperConfig()
        .jackson2ObjectMapperFactory((cls, charset) -> {
                    ObjectMapper objectMapper = new Jackson2ObjectMapperBuilder()
                            .serializerByType(MyRequest.class, new MyRequestSerializer())
                            .deserializerByType(MyResponse.class, new MyResponseDeserializer())
                            .featuresToDisable(...your features...)
                            .build();
                    return objectMapper;
                }
        ));
~~~
