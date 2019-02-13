---
layout: post
title: Kill specific process in windows batch
categories: Jenkins Windows
tags: jenkins ci windows process
---

CI 환경을 구성하다 보면, 실행되고 있는 어플리케이션을 내리고 새로 올리는 세팅을 빈번하게 하게 된다. 이 경우 특정 프로세스를 찾아서 종료시켜야 하는데, **Windows 환경에서** 간단한 스크립트로 특정 프로세스를 찾고 죽이는 방법에 대한 커맨드를 소개한다.

---

아래는 특정 포트로 서비스되는 프로세스를 찾아 종료시키는 커맨드이다.
~~~sh
FOR /F "tokens=5 delims= " %%P IN ('netstat -ano ^| findstr :8080 ^| findstr LISTENING') DO @echo taskkill /PID /F %%P
~~~

위의 스크립트에서 @echo를 지워주면 taskkill명령이 실제로 실행되니 주의. 설명하자면 netstat 명령으로 **LISTENING 상태의 포트 8080 사용하는** 프로세스 아이디를 얻어내고 taskkill /PID 명령으로 종료시킨다는 뜻이다.

---

아래와 같이 특정 프로세스 이름을 통해 종료시킬 수도 있다.

~~~sh
FOR /F "tokens=2 skip=3 delims= " %P IN ('tasklist /fi "imagename eq java*"') DO @echo taskkill /PID /F %P
~~~

위 커맨드는 **프로세스 이름이 java로 시작하는** 모든 프로세스ID를 찾아서 taskkill 명령으로 종료시킨다는 뜻이다.

커맨드에 대한 테스트는 bat 파일을 만들어서 cmd창에서 실행시켜 볼 수 있고, jenkins의 경우는 Execute Windows batch command 창에 바로 붙여넣으면 실행된다. 검색조건을 변경하여 활용하면 필요한 형태로 프로세스ID를 찾아내어 종료시킬 수 있다.
