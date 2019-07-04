---
layout: post
title: Stop all containers and remove all none tag docker images
categories: Docker
tags: docker ci
---

서비스를 제공하는 서버와 빌드서버가 분리되어 있고, 빌드서버에서 도커 이미지 빌드를 계속 하게 되면 태그가 없는 이미지가 많이 쌓이게 된다.
이런 경우 수작업으로 이미지 목록 내에서 `<none>` 태그로 표시된 이미지들을 지워줘도 되는데 이미지가 많이 쌓일수록 굉장히 번거로워진다. 이럴 때 간단한 스크립트로 사용하지 않는 `<none>` 태그 이미지를 한번에 삭제할 수 있다.

---
리눅스 서버의 경우
~~~sh
# stop all containers for linux
docker rm $(docker ps -a -q)

# remove all none tag images for linux
docker rmi $(docker images -q)
~~~
---
윈도우즈 서버의 경우
~~~sh
# stop all containers for windows
FOR /f "tokens=*" %i IN ('docker ps -a -q') DO docker rm %i

# remove all none tag images for windows
FOR /f "tokens=*" %i IN ('docker images -q -f "dangling=true"') DO docker rmi %i
~~~
