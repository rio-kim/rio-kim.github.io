---
layout: post
title: custom gradle plugin 만들기
categories: Gradle
tags: gradle plugin
---

gradle plugin을 단독 프로젝트로 만들어 보고 사용해본 후 까먹지 않기 위해 정리한다.

**우선 build.gradle에 다음과 같은 설정을 넣는다.**
~~~java
apply plugin: 'groovy'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

gradlePlugin {
  plugins {
    rioTask { // 사용하고자 하는 task 이름
        id = 'rio-plugin-id' // 플러그인id, 프로젝트명을 쓰면 되는듯
        implementationClass = 'com.mypackage.RioGradlePlugin'
                   // Plugin<T> 클래스를 상속받는 내가 직접 만든 클래스
    }
}
~~~

이후 프로젝트 내에 **Plugin&lt;T&gt;를 상속받는 클래스를 생성한다.** 이름과 위치는 상관없다. 예를 들어 RioPlugin이라고 가정한다.
~~~java
@Override
public void apply(Project project) {
  project.getTasks().create("rioTask", RioTask.class);
}
~~~
플러그인 클래스에는 이렇게 한 메서드만 있어도 된다. **Plugin&lt;T&gt;에서** 자동으로 오버라이드 되고, 내부는 저런식으로 쓰면된다. 간단히 설명하면 이 플러그인을 쓰는 곳에서는 task를 **rioTask라는** 이름으로 쓰게 되고 그 구현은 RioTask라는 클래스에 만든다.

그럼 사용하는 측에서는 이 플러그인을 사용할 때 이렇게 쓰게 되겠지.
~~~sh
gradle rioTask
~~~
느낌이 오겠지만 **RioTask라는 클래스에는 실제 task가 하는 일을 작성하는 곳이다.** 이 클래스는 **DefaultTask라는 클래스를 상속받고** 내부에 실제 로직이 들어갈 메서드를 작성한다.
~~~java
@TaskAction
public void rioTask() throws Exception {
  // 여기에 실제 로직이 들어간다.
}
~~~
실제로 만든 결과물만 보면 별로 어렵지 않게 만들 수 있다. 그런데 생각보다 리서치에도 시간이 걸렸고 삽질도 많이 했다. 안써놓으면 까먹을거 같은데 이거보면 금방 다시 만들수 있겠지.


##### 참고한 사이트

- [Writing Custom Plugins - Gradle User Guide Version 4.2.1](https://docs.gradle.org/current/userguide/custom_plugins.html)
- [Gradle Custom Plugins(권남위키)](http://kwonnam.pe.kr/wiki/gradle/customplugins)
