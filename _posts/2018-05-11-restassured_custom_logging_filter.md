---
layout: post
title: Custom logging filter in Rest-Assured
categories: Rest-Assured
tags: rest-assured request response log
---

보통 Rest-Assured를 이용해서 api를 테스트 할 때 로깅 세팅을 한다. Request와 Response를 테스트 로그로 남기며 테스트를 진행하는데, 대부분 아래와 같이 세팅한다.

~~~java
@Test
public void callGoogleBooksApi() {
    // https://www.googleapis.com/books/v1/volumes?q=flowers&filter=free-ebooks

    RequestSpecification requestSpecification = new RequestSpecBuilder()
            .setBaseUri("https://www.googleapis.com/books")
            .setBasePath("/v1")
            .setContentType(ContentType.JSON)
            .addFilter(new RequestLoggingFilter())
            .addFilter(new ResponseLoggingFilter())
            .build();

    given(requestSpecification)
            .queryParam("q", "flowers")
            .queryParam("filter", "free-ebooks")
            .get("volumes")
            .then().statusCode(200);
}
~~~

위와 같이 RequestSpecification 내부에 **addFilter 함수를 이용하여 RequestLoggingFilter와 ResponseLoggingFilter를 설정해준다.** 이 경우 Rest-Assured로 API 테스트를 진행하면 아래와 같이 console에 이쁘게 로그가 찍혀나온다.

![image1](https://user-images.githubusercontent.com/21053518/39916530-b720f1fc-5545-11e8-869c-47fb9311ff13.PNG)

필터 설정하는 부분만 바꾸어가며 여러가지 로깅 설정이 가능하다. **error가 있을 때만(status code가 400~500 사이의 경우) 전체 로그를 찍고 나머지 경우에는 로그가 나오지 않게끔 하기 위해 ErrorLoggingFilter를 사용하기도 한다.**

~~~java
RequestSpecification requestSpecification = new RequestSpecBuilder()
        .setBaseUri("https://www.googleapis.com/books")
        .setBasePath("/v1")
        .setContentType(ContentType.JSON)
        .addFilter(new ErrorLoggingFilter())
        .build();
~~~

이 경우에는, 시나리오상 호출하는 API 갯수가 많으면 테스트가 진행되는 동안 아무런 내용도 콘솔창에 찍히지 않기 때문에 답답함을 느낄 수도 있다. 그래서 심플한 로깅 필터를 사용해서 진행되는 내역을 콘솔에 찍어낼 수 있다. RequestLoggingFilter와 ResponseLoggingFilter 내에 파라미터로 들어가는 **LogDetail 설정을 지정하면 된다.**

~~~java
RequestSpecification requestSpecification = new RequestSpecBuilder()
        .setBaseUri("https://www.googleapis.com/books")
        .setBasePath("/v1")
        .setContentType(ContentType.JSON)
        .addFilter(new ErrorLoggingFilter())
        .addFilter(new RequestLoggingFilter(LogDetail.METHOD))
        .addFilter(new RequestLoggingFilter(LogDetail.URI))
        .addFilter(new ResponseLoggingFilter(LogDetail.STATUS))
        .build();
~~~

이 경우 실제 콘솔창에는 아래처럼 호출하는 api의 메서드 종류(GET, POST 등등..), uri, 그리고 response의 status code를 콘솔창에 출력해준다. 또한 ErrorLoggingFilter도 설정했으므로 error가 발생하는 경우 해당 api의 request, response에 대한 상세한 로그를 출력해준다.

![image2](https://user-images.githubusercontent.com/21053518/39916531-b747d89e-5545-11e8-9523-5e98bb8fcdb3.PNG)


진행했던 프로젝트에서는 하나의 테스트케이스를 돌리는 경우 20개 이상의 api가 시나리오대로 호출되는 경우들이 빈번했기 때문에 api 호출 1회당 3줄의 출력도 상당히 복잡하게 느껴졌다. 그래서 custom logging filter를 만들어 사용했다. 위와 같이 최소한으로 필요한 정보인 호출하는 api의 정보(uri, method)와 응답코드까지만 출력을 한줄로 끝내게 하기 위해 아래와 같은 방법을 활용했다.

~~~java
RequestSpecification requestSpecification = new RequestSpecBuilder()
        .setBaseUri("https://www.googleapis.com/books")
        .setBasePath("/v1")
        .setContentType(ContentType.JSON)
        .addFilter(new CustomRequestLoggingFilter())
        .addFilter(new CustomResponseLoggingFilter())
        .addFilter(new ResponseLoggingFilter(LogDetail.STATUS))
        .build();
~~~

아래는 custom logging filter 내부 구성

~~~java
public class CustomRequestLoggingFilter implements Filter {
    @Override
    public Response filter(FilterableRequestSpecification requestSpec, FilterableResponseSpecification responseSpec, FilterContext ctx) {
        System.out.print("[" + requestSpec.getMethod() + "]   " + requestSpec.getURI());
        return ctx.next(requestSpec, responseSpec);
    }
}

public class CustomResponseLoggingFilter implements Filter {
    @Override
    public Response filter(FilterableRequestSpecification requestSpec, FilterableResponseSpecification responseSpec, FilterContext ctx) {
        Response response = ctx.next(requestSpec, responseSpec);
        int statusCode = response.getStatusCode();
        if (statusCode >= 400 && statusCode <= 500) {
            System.out.println();
            ResponsePrinter.print(response, response, System.out, LogDetail.ALL, true);
        } else {
            System.out.println("  (" + response.getStatusCode() + ")");
        }
        return response;
    }
}
~~~

이 경우 아래처럼 api 1회 호출당 1줄의 콘솔 로그가 찍히고 오류가 나는 경우는 ErrorLoggingFilter 적용한 것과 같이 전체 로그가 찍히게 된다.

![image3](https://user-images.githubusercontent.com/21053518/39916532-b76e131a-5545-11e8-9281-5cd05bac14c6.PNG)

상황에 맞게 필요한 로그를 사용하면 될 듯 하고, 많은 상황에서는 첫번째 전체 request와 response를 출력하는 설정을 쓰곤 한다.
