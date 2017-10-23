---
layout: post
title: 80port로 linux에서 tomcat 서버 올리기
categories: Linux
tage: linux tomcat
---

리눅스의 경우 80포트로 서버를 띄우기 위해서는 root계정(sudo) 명령을 사용해야 한다.
그래서 서비스를 단독으로 실행하지 않는 경우 띄우기가 힘들어서 tomcat에서 내부적으로 실행하는 war파일은 80포트를 이용하기 힘들다.

그래서 다음의 방법을 사용한다.
우선 tomcat에서 기본적으로 사용하는 8080포트를 서버로 띄운 후, 외부에서 들어오는 80포트를 리눅스 내부적으로 리다이렉트로 8080포트로 연결 시켜주는 것.
다시말해 실제 서버는 8080포트에 떠있고 외부에서 80포트로 접속을 하면 8080포트로 알아서 연결 해준다.

다음의 명령어를 통해 기본적인 tomcat 8080포트를 80포트에 연결할 수 있다.

     sudo iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
     sudo service iptables save
