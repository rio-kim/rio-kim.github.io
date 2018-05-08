---
layout: post
title: Font ligatures in IntelliJ
categories: IntelliJ
tags: intellij plugin editor tip
---

IntelliJ 에디터 사용시 폰트 합자(?) 적용하는 방법에 대해서 적어본다. 좀 쓸데없는 팁이지만, 괜히 이쁘게 쓰는 방법 같아서 남겨봄.

**Settings > Editor > Font > Enable font ligatures 체크**

아래와 같이 연산자나 특수기호 등이 에디터에서 예쁘게 표시되게끔 바꾸어준다. 특히 ==, !=, >= 등의 비교 연산자나 &&, || 등의 연결부호, 람다식에서의 arrow function(->) 등이 예쁘게 표시된다.

##### Enable font ligatures 체크 해제(적용 전)
![not_check](https://user-images.githubusercontent.com/21053518/39746583-b0e58d50-52e5-11e8-888a-8c43dfe34871.PNG)

##### Enable font ligatures 체크(적용 후)
![check](https://user-images.githubusercontent.com/21053518/39746586-b17f6dc6-52e5-11e8-82ed-946fb2fb623b.PNG)

자세히 보기 전에는 잘 모르는 부분들도 바뀌는데 주석 기호(//), 더하기2개(++), www 글씨, 복수 개의 파라미터를 받는 점 세개 varargs(...) 등이 바뀌었다. 기능적인 부분에는 전혀 변화가 없는, 팁이라고 하기도 좀 애매한 팁이지만 뭔가 예쁘게 인텔리제이를 쓰는 느낌이 듬.
