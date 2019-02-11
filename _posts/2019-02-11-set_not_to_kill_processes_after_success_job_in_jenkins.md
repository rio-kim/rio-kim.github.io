---
layout: post
title: Set not to kill processes after success job in Jenkins
categories: Jenkins
tags: jenkins ci windows
---

jenkins로 ci환경을 구성한 후 서버를 올리는 job을 만들고 Build Now로 수행시키면 서버는 정상적으로 올라가나 job이 끝나지 않는다.
예를들어

~~~sh
java -jar app.jar
~~~

위의 명령이 젠킨스 job에서 수행이 된다면 app.jar는 정상적으로 올라가나 수행된 job은 사용자가 강제 종료하기 전까지 끝나지 않는다.

이 경우 linux 계열 위에서 올라간 jenkins에서는 **nohup** 명령어를 통해 간단히 job을 종료하면서 프로세스를 계속 살려둘 수 있다.

윈도우에서도 비슷한 역할을 하는 명령어가 있다.

~~~sh
start /min java -jar app.jar
~~~

start 명령만 사용하면 cmd창이 새로 뜨는데 **/min** 옵션을 추가하면 cmd창 없이 프로세스만 계속 지속시킬 수 있다.

만약 이렇게 설정해도 job 종료와 동시에 프로세스가 죽는다 하면 jenkins 서비스를 띄울때 **-Dhudson.util.ProcessTree.disable=true** 아규먼트를 추가해서 실행해보자.

~~~sh
java -Dhudson.util.ProcessTree.disable=true -jar jenkins.war
~~~

만일 윈도우 설치 패키지(msi)로 jenkins를 설치하여 윈도우 서비스에 올라가 있는 상황이라면 jenkins 설치 폴더를 찾아 폴더 내에 **jenkins.xml** 파일을 수정한다. 수정할 위치는 xml 파일 내에서 service > arguments 부분을 찾아서 적당한 위치에 위의 옵션을 넣어준다.

~~~xml
<service>
  <arguments>이곳에 java -jar로 젠킨스 서비스 올리는 명령이 들어가 있다</arguments>
</service>
~~~

xml 파일을 수정한 후 윈도우의 서비스 관리 창을 열어 jenkins 서비스를 찾아 다시 시작한다.


##### 참고한 사이트

- [Jenkins WIKI - ProcessTreeKiller](https://wiki.jenkins.io/display/JENKINS/ProcessTreeKiller)
