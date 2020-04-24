---
layout: post
title: Kill process by port and nohup in ubuntu jenkins
categories: Jenkins
tags: jenkins ci ubuntu
---
굉장히 간단해보이는데 삽질을 오래해서 까먹지 말라고 적어둔다.

1. **ubuntu jenkins**에서
2. **ssh plugin**을 이용해 다른 **ubuntu**로 접속 후
3. 실행중인 process를 port로 검색해서 죽인 후
4. nohup으로 새로 띄우는 스크립트이다.

~~~sh
# 1. port(여기서는 5000포트)를 쓰는 processid를 찾아 변수 할당한다.
pid="$(lsof -t -i :5000 -s TCP:LISTEN)";

# 2. processid가 있는 경우에 kill 명령으로 중지시킨다.
if [ "$pid" != "" ]; then
  kill -9 $pid;
  echo "$pid process kill complete"
else
  echo "pid is empty"
fi

cd ~/git/petopia-crawler
# 3. 원하는 새 프로세스를 nohup으로 실행시킨다.
BUILD_ID=dontKillMe DISPLAY=:0 nohup ~/anaconda3/envs/petopia-crawler/bin/python3.7 run.py >> nohup.out 2>&1 &
~~~
