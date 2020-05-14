---
layout: post
title: yolo v3 custom data train
categories: yolo
tags: yolo train deeplearning
---

##### 1. AlexeyAB darknet 모델 다운로드
~~~sh
https://github.com/AlexeyAB/darknet.git
cd darknet
~~~

##### 2. 빌드를 위해 Makefile 수정 후 빌드
- 내가 수정한 부분
~~~
CPU=1
CUDNN=1
OPENCV=1
~~~
~~~sh
make
~~~

##### 3. 테스트
~~~sh
./darknet detector test cfg/coco.data cfg/yolov3.cfg yolov3.weights data/dog.jpg
~~~

##### 4. 학습을 위한 사전 파일 준비
~~~sh
custom_data/custom.names
custom_data/images
custom_data/train.txt
custom_data/test.txt
~~~

##### 4-1. custom.names: 클래스명을 나열한 파일
~~~
none
stand-four-legs
stand-two-legs
sit-down
kneel-down
play-bow
eating
peeing
pooping
lay-belly
lay-curled
lay-side
lay-sprawled
~~~

##### 4-2. images 폴더
- 이미지파일과 annotation txt파일이 같은 파일명으로 존재해야 한다.
- 예를들면 a.jpg, a.txt가 세트로 존재해야 한다.

##### 4-3. annotatino txt파일 형식
~~~
<object-class-id> <x-center> <y-center> <width> <height>
~~~
- 여기서 x-center, y-center는 그림에서 사각형 박스의 중심좌표이다.
- width, height는 사각형 좌표의 크기를 나타낸다.
- 이 모든 것은 전체 이미지 사이즈의 비율로 소수점 처리가 되어야 한다.
- x-center, y-center는 0과 1 사이의 값이고 0, 1과 같을 수 없다.
- width, height는 0, 1도 가능하고 0과 1사의 값을 가진다.

~~~
0 0.1242 0.63632 0.123 0.5
~~~

##### 4-4. train.txt와 test.txt파일
- 이미지 파일 목록을 나타낸다. 두 파일 다 아래와 같은 형태로 나타낸다.

~~~
custom_data/images/a.jpg
custom_data/images/b.jpg
custom_data/images/c.jpg
...
~~~
#### 4-5. detector.data 파일
~~~
classes=1
train=custom_data/train.txt
valid=custom_data/test.txt
names=custom_data/custom.names
backup=backup/
~~~

- classes: 클래스 갯수
- train, valid: 이미지 목록 파일
- names: 클래스 목록 파일
- backup: 트레이닝된 모델이 순차적으로 쌓이는 폴더 경로
- 주의할 점은 backup 폴더는 darknet root에 만들어 놓고 트레이닝을 시작해야 한다. 폴더가 없으면 트레이닝 중 에러가 나는 듯.

##### 5. cfg 파일 작성
- ./cfg 폴더 밑에 있는 yolov3.cfg 파일을 복사해서 custom_data/cfg/yolov3-custom.cfg 파일을 만든다.
- 해당 파일을 열어 아래와 같이 수정한다.
- batch `line: 6`
  - 기본값 64. 건드리지 않는다.
- subdivisions `line: 7`
  - 배치 사이즈를 얼마나 쪼개서 학습할지 설정한다. 기본값은 8이지만 메모리 릭이 나면 16 32 등으로 조절한다.
- witdh, height `line: 8, 9`
  -  기본값은 416, 608로 변경하면 정확도가 좋아질 수 있다. 32의 배수로 조절한다.
- learning_rate `line: 18`
  - 기본값 0.001, multi gpu인 경우 0.001/gpu수로 설정한다. 예를들어 2개라면 0.0005를 써준다.
- burn_in `line: 19`
  - 기본값 1000, multi gpu인 경우 1000*gpu수로 설정한다. 예를들어 2개라면 2000을 써준다.
- max_batches `line: 20`
  - 이터레이션 갯수, class수*2000+200. 예를들어 5개 클래스라면 10200
- steps `line: 22`
  - max_batches의 80%, 90%로 class갯수*2000의 80%, 90%를 설정한다. 예를들어 5개 클래스라면 8000,9000을 써준다.
- classes `line: 610, 696, 783`
  - [yolo] 레이어에 있는 것으로 클래스 수를 적어준다. 예를들어 5개 클래스라면 5를 써준다.
- filters: `line: 603, 689, 776`
  - (클래스 갯수+5)*3을 써준다. 예를들어 5개 클래스라면 30을 써준다.
  - 문자열 검색을 하면 많이 나오는데 **yolo레이어 바로 위에 있는 필터만 수정한다.**

##### 6. 트레이닝 시작
~~~sh
# ./darknet detector train {data파일} {cfg파일} {trainset 파일} {gpu갯수 옵션}
./darknet detector train custom_data/cfg/detector.data custom_data/cfg/yolov3-custom.cfg darknet53.conv.74 -gpus 0,1
# nohup 실행
nohup ./darknet detector train custom_data/cfg/detector_yolo_rio.data custom_data/cfg/yolov3-custom_yolo_rio.cfg darknet53.conv.74 -gpus 0,1 &
~~~
##### 7. 트레이닝 후 로그 보는 방법

- v3로 시작하는 라인들이 계속 나오는데 중간에 이터레이션 번호가 뜨는 라인이 생김
- 이터레이션 번호 뒤 3열에 avg loss값이 있는데 그게 중요하다. 아래와 같은 형태로 나옴

~~~
 10: 1630.738525, 1685.839600 avg loss, 0.000000 rate, 4.303994 seconds, 1280 images, 10.956437 hours left
...
 186: 714.564453, 1032.841675 avg loss, 0.000000 rate, 4.592934 seconds, 23808 images, 8.222052 hours left
...
 700: 1.113163, 1.076701 avg loss, 0.000015 rate, 4.974880 seconds, 89600 images, 6.471217 hours left
...
 902: 0.889429, 0.902189 avg loss, 0.000041 rate, 4.518785 seconds, 115456 images, 6.284531 hours left
...
~~~

- avg loss가 위의 경우 10 이터레이션의 1685.839600 값이 902 이터레이션에서 0.902189까지 떨어졌다.
- iteration 이 증가할 수록 avg loss 값이 떨어져야 하고 어느정도 떨어지면 더 잘 안떨어진다.
- 데이터 세트 숫자에 따라서 떨어지는 값이 다른데 적을때는 0.00x까지도 떨어진다.
- 맨 뒤에 남은 시간이 표시된다.

##### 8. 트레이닝 종료 후 모델 테스트
- 아까 만든 yolov3-custom.cfg 파일을 복사해서 yolov3-custom-test.cfg 파일을 만든다.
- 안에 내용을 아래와 같이 수정한다. 3, 4 라인 주석 해제 후 6, 7라인을 주석처리한다.

~~~
[net]
# Testing
batch=1
subdivisions=1
# Training
# batch=64
# subdivisions=16
width=608
height=608
~~~

##### 8-1. 이미지 테스트
~~~sh
# ./darknet detector test {data파일} {테스트cfg파일} {학습한weights파일} {테스트할이미지}
./darknet detector test custom_data/detector.data custom_data/cfg/yolov3-custom-test.cfg backup/yolov3-custom_last.weights ./test.png
~~~

- 이미지 결과가 창으로 뜨거나 원격ssh 환경 같은 경우에는 predictions.jpg 파일로 떨어진다.

##### 8-2. 비디오 테스트
~~~sh
# ./darknet detector demo {data파일} {테스트cfg파일} {학습한weights파일} {테스트할비디오} {-옵션}
./darknet detector demo custom_data/detector.data custom_data/cfg/yolov3-custom-test.cfg backup/yolov3-custom_last.weights ./test.video -dont_show -out_filename out.avi
~~~

- -dont_show: 원격 ssh 환경 등에서 비디오 오픈 없이 실행한다.
- -out_filename out.avi: 결과 비디오를 out.avi파일로 저장한다.

##### 9. 트레이닝 종료 후 모델 정확도 측정

~~~sh
# ./darknet detector map {data파일} {cfg파일} {측정하려는weights파일}
./darknet detector map custom_data/detector.data custom_data/cfg/yolov3-custom.cfg backup/yolov3-custom_last.weights
~~~

- 중간 생략하고 결과가 아래와 같이 나온다.

~~~
 detections_count = 628, unique_truth_count = 278
class_id = 0, name = none, ap = 14.65%           (TP = 5, FP = 9)
class_id = 1, name = stand-four-legs, ap = 82.66%        (TP = 77, FP = 38)
class_id = 2, name = stand-two-legs, ap = 100.00%        (TP = 2, FP = 0)
class_id = 3, name = sit-down, ap = 75.25%       (TP = 35, FP = 23)
class_id = 4, name = kneel-down, ap = 68.33%     (TP = 21, FP = 27)
class_id = 5, name = play-bow, ap = 0.00%        (TP = 0, FP = 1)
class_id = 6, name = eating, ap = 73.92%         (TP = 46, FP = 20)
class_id = 7, name = peeing, ap = 0.00%          (TP = 0, FP = 0)
class_id = 8, name = pooping, ap = 0.00%         (TP = 0, FP = 0)
class_id = 9, name = lay-belly, ap = 29.17%      (TP = 0, FP = 0)
class_id = 10, name = lay-curled, ap = 0.00%     (TP = 0, FP = 0)
class_id = 11, name = lay-side, ap = 25.00%      (TP = 1, FP = 2)
class_id = 12, name = lay-sprawled, ap = 55.00%          (TP = 2, FP = 0)

 for conf_thresh = 0.25, precision = 0.61, recall = 0.68, F1-score = 0.64
 for conf_thresh = 0.25, TP = 189, FP = 120, FN = 89, average IoU = 52.19 %

 IoU threshold = 50 %, used Area-Under-Curve for each unique Recall
 mean average precision (mAP@0.50) = 0.403062, or 40.31 %
Total Detection Time: 21 Seconds
~~~

- precision: TP/(TP+FP), 맞춘 수/예측한 전체 모수
- recall: TP/(TP_FN), 맞춘 수/어노테이션 수
- IoU: 교집합/합집합 면적
- 목표는 precision, recall은 0.8정도까지, mAP@0.50은 0.6정도까지
- 결과 분석은 아래 블로그 참조
  - https://hoya012.github.io/blog/Tutorials-of-Object-Detection-Using-Deep-Learning-how-to-measure-performance-of-object-detection
