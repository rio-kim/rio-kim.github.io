---
layout: post
title: REST API client library in Java project
categories: RestAPI
tags: rest api library java
---

JAVA 프로젝트에서 REST API를 호출하는 방법은 여러 가지가 있다. 예제를 통해 기록한다. 예제는 bithumb 홈페이지에서 현재 비트코인 시세를 조회하는 open api를 호출하여 json형태의 결과를 가져오는 방법이다.

1. **JAVA**
우선 가장 기본적인 방법은 **http connection을 열고 api를 호출한 후** 결과를 받아오는 방법이다. 이 방법은 외부 라이브러리를 쓰지 않고 **순수한 java라이브러리만 활용한다.** 사실 요새 외부 라이브러리들이 엄청 잘 나와 있는 시점에 굳이 이렇게 쓸 일은 거의 없을 것이지만, 기록을 위해...

    ##### Pure Java code example
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

2. **RestTemplate**
Spring 프로젝트에서는 일반적으로 RestTemplate을 활용한다. RestTemplate은 스프링 기반 프로젝트 구성 하에서 기본적으로 제공한다. 내가 만약 Spring 기반의 프로젝트를 구성하고 있고 별다른 추가 기능 없이 REST Api 호출 및 response 값 활용 정도의 목적이면 여타 다른 라이브러리를 사용하기보단 그냥 이걸 쓰는걸 추천.

    ##### RestTemplate example
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

3. **Rest-Assured**
이 라이브러리는 response에 대한 검증에 특화된 특히 테스팅에 많이 쓰이는 라이브러리이다. 그렇다 해도 기본적인 REST API 호출 및 결과값에 대한 핸들링이 기타 다른 라이브러리들에 비해 상당히 편하다고 느꼈다.
    ##### Rest-Assured example
    ~~~java
    // TODO
    // 작성중
    ~~~


4. Retrofit

5. Unirest
