---
layout: post
title: tomcat war배포시 logback 설정
categories: Linux
tags: linux tomcat logbak
---

logback설정시 log file이 생기는 위치를 permission 접근 가능한 절대 경로 위치로 해주는 것이 좋다.

개발 환경에서는(특히 Windows) log file 위치를 아무곳에 떨구어도 springboot가 올라가면서 알아서 생성해 주는데 linux환경에서 경로 접근 권한이 없으면 tomcat으로 띄워진 서버에 접속해도 별다른 반응 없이 브라우저에 연결 지연만 떠서 매우 고생ㅠㅠ

이 경우 application.yml 파일 내에 local(윈도우) 환경과 서버(리눅스) 환경을 나누어 설정 해주는 것이 좋다. 우선 **logback-local.xml, logback-production.xml** 등 두 가지 버전으로 log file을 생성해주는 설정을 만든 후 application.yml에 들어가서 기본 설정 맨 밑에 **spring.profiles** 옵션을 이용하여 프로파일을 분리해준다.

tomcat 설정에서도 active profile 설정이 필요한데, 아래와 같이 파일을 만들면 tomcat이 시작할 때 지정된 옵션으로 환경변수를 넣어 실행한다.


##### /usr/share/tomcat8/bin/setenv.sh 파일 생성(또는 편집)

    export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=prod"


이후 tomcat을 재시작한다.

    sudo service tomcat8 restart


##### 참고) tomcat 서비스 로그 확인

     sudo tail -200f /usr/share/tomcat8/logs/catalina.out
