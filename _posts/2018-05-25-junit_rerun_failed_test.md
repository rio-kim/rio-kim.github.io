---
layout: post
title: JUnit rerun failed test
categories: Testing
tags: test junit
---

프로젝트 진행중에 자동화 테스트를 수행하다 보면, 테스트 코드 또는 프로덕트 코드상의 로직 문제 외에도 실패하는 경우들이 발생하곤 한다. 불안정한 네트워크 환경, 테스트에 종속된 서비스가 내려가 있는 경우 등등 여러가지 이유가 있다. 이런 실패는 특히 개발 단계의 테스트 수행시 더욱 빈번하다. **또 GUI 자동화 테스트의 경우에는 더 다양한 경우들이 발생한다. 브라우저의 문제 또는 수행하는 Local PC에서 예상치 못한 화면 전환이 있다거나 더욱 다양한 이유로 인해 코드의 문제와는 상관없이 외부 요인으로 테스트가 깨지는 경우들이 발생한다.**

이유야 어쨌든 테스트가 실패하면 실패한 테스트만 모아서 다시 재수행해보는 경우들이 생기고, 그런 상황에서 junit rule을 활용하는 방법이 있다. 테스트 클래스 내에 Rule 어노테이션을 활용하여 다음과 같이 Retry Rule을 적용한다.

~~~java
@Rule
public RetryRule retryRule = new RetryRule(RETRY_COUNT);
~~~

RetryRule이라는 클래스를 만들어 준다. 생성자 안에 파라미터로 count를 받는 형태로 만들었다. 형태는 아래와 같으며 필요에 따라 조금씩 변형해서 사용하면 될 듯 하다.

~~~java
public class RetryRule implements TestRule {

    private int retryCount;

    RetryRule(int retryCount) {
        this.retryCount = retryCount;
    }

    @Override
    public Statement apply(Statement base, Description description) {
        return statement(base, description);
    }

    private Statement statement(final Statement base, final Description description) {
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                Throwable caughtThrowable = null;

                for (int i = 0; i < retryCount; i++) {
                    try {
                        base.evaluate();
                        return;
                    } catch (Throwable t) {
                        t.printStackTrace();
                        caughtThrowable = t;
                        System.err.println(description.getDisplayName() + ": run " + (i + 1) + " failed.");
                    }
                }
                System.err.println(description.getDisplayName() + ": giving up after " + retryCount + " failures.");
                throw caughtThrowable;
            }
        };
    }
}
~~~

이렇게 세팅 후 JUnit 테스트를 수행하면 **실패가 있는 경우 설정한 count만큼 재수행을 시도한다. 그리고 재수행 하면서 성공한 경우에는 테스트를 성공한 것으로 간주한다.** junit 리포트에서도 성공한 테스트로 간주하니 염두해 두고 사용하면 된다.

테스트가 코드 외의 환경에 영향을 받는 경우이면서 외부 환경의 일시적인 문제로 실패한 경우 대부분 재수행하여 성공을 원하는 경우엔 이런 룰을 적용하며 테스트를 수행한다면 많은 도움을 받을 수 있다. 하지만 사이드 이펙트가 있을 수 있는데 **통제되어야 하는 외부 환경의 영향으로 간헐적 실패가 있는 경우에는 이런식으로 해결하는 접근 보다는 외부 환경을 mocking 하거나 독립적인 테스트 환경을 구성하는 쪽으로 고려하는 편이 옳은 방법이라 볼 수 있겠다.**
