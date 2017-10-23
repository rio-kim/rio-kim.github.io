---
layout: post
title: gradle testset을 분리하여 관리하기
categories: Gradle
tags: gradle test plugin
---

기본적으로 그래들 프로젝트는 unit test를 **main/test/java** 형태로 두는데, 이 외에 다른 test set을 구성하고 싶을 때가 있다. 예를 들면 같은 프로젝트 내에 api test를 따로 구성한다거나, ui test를 구성한다거나.

이런 경우에 test를 분리하는 방법에는 여러가지가 있는데 **gradle-testsets-plugin** 플러그인을 통해서 하는 방법을 알아보자.

build.gradle 설정

    plugins {
      id 'org.unbroken-dome.test-sets' version '1.4.2'
    }

또는 하위 버전에는 이런 식으로 플러그인을 추가한다.

    buildscript {
        repositories {
            jcenter()
        }
        dependencies {
            classpath 'org.unbroken-dome.gradle-plugins:gradle-testsets-plugin:1.4.2'
        }
    }

    apply plugin: 'org.unbroken-dome.test-sets'


testset을 분리하기 위해 아래의 명령도 추가한다.

    testSets {
      integrationTest
      integrationOfIntegrationTest { extendsFrom integrationTest }
    }

만약 위와같이 쓴다면 testSet을 두개 더 만든거다.

따로 설정하지 않는다면 src/test/java 형태처럼 src/integrationTest/java
거고 상속관계도 저런식으로 처리할수 있다. integrationTest에 있는 놈들을 integrationOfIntegrationTest 쪽에서 쓸수 있다.

**testSets 밑에 들어가는 새로만든 testSet은 기본 test를 자동 상속받는다.**

만일 integrationTest 에서만 쓰는 외부 라이브러리(ex. rest-assured)가 있다면 아래와 같이 설정한다.

    dependencies {
      integrationTestCompile group: 'io.rest-assured', name: 'rest-assured', version: '3.0.3'
    }

- [plugin 공식 홈페이지: unbroken-dome/gradle-testsets-plugin](https://github.com/unbroken-dome/gradle-testsets-plugin)
