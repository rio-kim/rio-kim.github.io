---
layout: post
title: test sourceset in gradle project
categories: Testing
tags: gradle test plugin integrationTest
---

저번에 작성했던 ["gradle testset plugin"](https://rio-kim.github.io/testing/2017/10/23/gradle_testset_plugin)에서 기본적으로 제공되는 테스트셋인 **src/test/java** 형태 외의 새로운 테스트 셋을 만드는 방법을 정리했는데, 그 플러그인을 사용하는 경우 한 가지 문제점이 발생한다.

보통 프로젝트 상황에서 main sourceset에 있는 모델 객체를 unit test에서 사용하기 위해 모델 객체의 builder등을 만들어 사용하곤 한다. 이런 경우에 **gradle-testsets-plugin** 플러그인을 사용하면 **src/test/java** 내에 있는 builder 또는 테스트를 위해 만든 util등을 사용할 수 없다. 다시말해 내가 새로이 만든 testset에서 **src/test/java** 내에 있는 코드를 사용할 수 없다는 말이다.(다행히 이 경우에도 build.gradle 내의 dependencies는 상속해서 사용할 수 있다. 또한 main sourceset에 있는 코드는 사용할 수 있다)

여튼, testset 내에서 작성한 코드를 새로 만든 testset에서 사용하기 위해서는 아래와 같이 설정하면 된다. gradle-testsets-plugin를 사용하지 않는다. 예시를 위해 **integrationTest** 이름을 가진 testset을 만든다고 가정한다.

build.gradle 설정
~~~java
// gradle sourceset 설정
sourceSets {
    integrationTest {
        java {
            compileClasspath += main.output + test.output
            runtimeClasspath += main.output + test.output
            srcDir file('src/integrationTest/java')
        }
        resources.srcDir file('src/integrationTest/resources')
    }
}

// gradle task 설정
task integrationTest(type: Test) {
    testClassesDirs = sourceSets.integrationTest.output
    classpath = sourceSets.integrationTest.runtimeClasspath
}

// testset을 상속받겠다는 설정
configurations {
    integrationTestCompile.extendsFrom testCompile
    integrationTestRuntime.extendsFrom testRuntime
}

~~~
위와 같이 설정하면 **src/test/java** 내에서 작성한 코드들을 **src/integrationTest/java** 내에서 import하고 사용할 수 있다.

- [gradle docs: SourceSet](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.SourceSet.html)
- [gradle docs: Configuration](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html)
- [gradle docs: Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html)
