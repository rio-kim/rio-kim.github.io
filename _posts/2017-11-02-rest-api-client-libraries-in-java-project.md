---
layout: post
title: REST API client library in Java project
categories: RestAPI
tags: rest api library java
---

JAVA 프로젝트에서 REST API를 호출하는 방법은 여러 가지가 있다. 예제를 통해 기록한다. 예제는 bithumb 홈페이지에서 현재 비트코인 시세를 조회하는 open api를 호출하여 json형태의 결과를 가져오는 방법이다.

### 1. Pure JAVA Code
우선 가장 기본적인 방법은 **http connection을 열고 api를 호출한 후** 결과를 받아오는 방법이다. 이 방법은 외부 라이브러리를 쓰지 않고 **순수한 java라이브러리만 활용한다.** 사실 요새 외부 라이브러리들이 엄청 잘 나와 있는 시점에 굳이 이렇게 쓸 일은 거의 없을 것이지만, 기록을 위해.
~~~java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class CallRestBithumbOpenApi {

    public String callTickerBtc() throws Exception {

        String urlString = "https://api.bithumb.com/public/ticker/btc";
        URL url = new URL(urlString);
        HttpURLConnection con = (HttpURLConnection) url.openConnection();
        con.setRequestMethod("GET");

        InputStreamReader in = new InputStreamReader(con.getInputStream(), "utf-8")
        BufferedReader br = new BufferedReader(in);

        String line;
        StringBuilder sb = new StringBuilder();
        while ((line = br.readLine()) != null) {
            sb.append(line).append("\n");
        }
        br.close();
        return sb.toString();
    }
}
~~~

### 2. RestTemplate
Spring 프로젝트에서는 일반적으로 RestTemplate을 활용한다. RestTemplate은 스프링 기반 프로젝트 구성 하에서 기본적으로 제공한다. 내가 만약 Spring 기반의 프로젝트를 구성하고 있고 별다른 추가 기능 없이 REST Api 호출 및 response 값 활용 정도의 목적이면 여타 다른 라이브러리를 사용하기보단 그냥 이걸 쓰는걸 추천.
~~~java
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

public class CallRestBithumbOpenApiRestTemplate {

    public Map callTickerBtc() throws Exception {

        // json형태의 return type인 경우 필요한 형태로 response를 재구성할 수 있다.
        // 이 경우는 map형태로 치환하여 받아왔다.
        RestTemplate restTemplate = new RestTemplate();

        String url = "https://api.bithumb.com/public/ticker/btc";
        ResponseEntity<Map> rawResult
                = restTemplate.getForEntity(url, Map.class);
        Map<String, Object> response = rawResult.getBody();
        return response;
    }
}
~~~

근데 모든 프로젝트가 스프링은 아니니까, RestTemplate를 쓰기 힘든 경우가 있다. 요것만 쓰기 위해 프로젝트 구성에 Spring dependency를 추가할 필요는 없다. 다른 잘 만들어진 오픈소스 라이브러리들이 있기 때문이다. 종류는 아래와 같다.

### 3. Rest-Assured
이 라이브러리는 response에 대한 검증에 특화된 특히 테스팅에 많이 쓰이는 라이브러리이다. 그렇다 해도 기본적인 REST API 호출 및 결과값에 대한 핸들링이 기타 다른 라이브러리들에 비해 상당히 편하다고 느꼈다. 아래의 예시는 가장 간단한 방식으로 response value를 얻고자 할 때 사용할 수 있으며 모델링 객체로 변환하거나 전체 response 내에서 원하는 값만 얻고자 할 때에는 jsonPath를 활용하여 사용할 수 있다. 이는 나중에 다시 포스트를 올려볼 예정.
~~~java
import io.restassured.RestAssured;

import java.util.Map;

public class CallRestBithumbOpenApiRestAssured {

    public Map callTickerBtc() throws Exception {

        String url = "https://api.bithumb.com/public/ticker/btc";
        Map<String, Object> response = RestAssured.get(url).jsonPath().<Map>get("data");
        return response;
    }
}
~~~

### 4. Retrofit
Retrofit은 조금 다른 방식으로 작성된다. 다른 라이브러리와 가장 구별되는 특징은 API를 인터페이스 형태로 관리하여 사용하는 측에서 인터페이스를 호출하는 듯한 느낌을 주게 하는 것이다. 많은 갯수의 API들을 호출해야 하는 프로젝트에서 사용할 때 관리의 용이성 면에서 장점을 가질 수 있으나, 반대로 API를 호출하기 위해 작성하는 코드의 양이 다른 라이브러리를 쓸 때보다 많아진다.
- Interface 작성
~~~java
import retrofit2.Call;
import retrofit2.http.GET;

import java.util.Map;

public interface BithumbApiService {

    @GET("/public/ticker/btc")
    Call<Map> callBtc();
}
~~~

- 호출하는 코드 작성
~~~java
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.converter.jackson.JacksonConverterFactory;

import java.util.Map;

public class CallRestBithumbOpenApiRetrofit {

    public Map callTickerBtc() throws Exception {

        // Retrofit 전체 설정은 처음 한 번만 선언해 주면 된다.
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://api.bithumb.com")
                .addConverterFactory(JacksonConverterFactory.create())
                .build();

        BithumbApiService service = retrofit.create(BithumbApiService.class);
        Response<Map> execute = service.callBtc().execute();
        Map<String, Object> response = execute.body();

        return response;
    }
}
~~~

### 5. Unirest
아주 lightweight한 라이브러리라고 자신을 소개하고 있다. 사용법은 간단하다. 특징은 response를 순수한 JSONObject 형태로 리턴하기가 매우 쉽다는 점. 이렇게 순수한 형태의 JSONObject를 서비스단에서 핸들링할 일이 많으면 이 라이브러리를 썼을 때 이점을 얻을 수 있을 것이다.
~~~java
import com.mashape.unirest.http.HttpResponse;
import com.mashape.unirest.http.JsonNode;
import com.mashape.unirest.http.Unirest;
import org.json.JSONObject;

public class CallRestBithumbOpenApiUnirest {

    public JSONObject callTickerBtc() throws Exception {

        String url = "https://api.bithumb.com/public/ticker/btc";
        HttpResponse<JsonNode> response = Unirest.get(url).asJson();
        return response.getBody().getObject();
    }
}
~~~
작성하다보니 get메서드를 기본 예제로 하다보니 post에서 request를 만들어 넘기는 방식에 대한 언급이 좀 없게 되어서 아쉽지만 특히 라이브러리를 사용하는 경우에는 사용법이 그렇게 어렵지 않다. 아래의 사이트를 보면서 쉽게 따라할 수 있다.

이상의 오픈소스 라이브러리를 써보면서 느낀 점은, 라이브러리를 선택할 때 프로젝트의 상황에 따라 적절한 라이브러리가 다 다를것 같다는 생각이다. 개인적으로는 기능도 많고 사용을 많이 해봐서 익숙한 rest-assured를 자주 쓰게 되긴 함.

##### 각 라이브러리 사이트
- [RestTemplate Javadoc](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)
- [Rest-Assured](http://rest-assured.io/)
- [Rest-Assured User Guide](https://github.com/rest-assured/rest-assured/wiki/Usage)
- [Retrofit](http://square.github.io/retrofit)
- [Retrofit 한글문서](http://devflow.github.io/retrofit-kr)
- [Unirest for java](http://unirest.io/java.html)
