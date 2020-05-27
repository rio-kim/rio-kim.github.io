---
layout: post
title: Install opencv in ubuntu
categories: opencv
tags: opencv ubuntu
---

conda 환경에서 opencv를 설치했던 기록을 남긴다.

##### 1. 기존 설치 확인

~~~sh
pkg-config --modversion opencv
~~~

버전이 출력되지 않고 `No package 'opencv' found`가 떠야한다.

##### 2. 기존 버전이 설치되어 있으면 삭제한다.

~~~sh
# 기존 라이브러리 설정파일 및 패키지 삭제
sudo apt-get purge  libopencv* python-opencv
sudo apt-get autoremove

#기존설치된 opencv 라이브러리 삭제
sudo find /usr/local/ -name "*opencv*" -exec rm -i {} \;
~~~

##### 3. 설치준비

~~~sh
sudo apt-get update
sudo apt-get upgrade
~~~

##### 4. opencv 컴파일 위해 필요한 패키지 설치

~~~sh
sudo apt-get install build-essential cmake pkg-config libjpeg-dev libtiff5-dev libpng-dev libavcodec-dev libavformat-dev libswscale-dev libxvidcore-dev libx264-dev libxine2-dev libv4l-dev v4l-utils libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgtk2.0-dev libgtk-3-dev mesa-utils libgl1-mesa-dri libgtkgl2.0-dev libgtkglext1-dev libatlas-base-dev gfortran libeigen3-dev python2.7-dev python3-dev python-numpy python3-numpy python3.5-dev
~~~

##### 5. 4.2.0버전 다운로드

~~~sh
mkdir opencv
cd opencv
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.2.0.zip
unzip opencv.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.2.0.zip
unzip opencv_contrib.zip
~~~

##### 6. 빌드를 위한 디렉토리 생성

~~~sh
cd opencv-4.2.0
mkdir build
cd build
~~~

##### 7. 빌드

~~~sh
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=$(python -c "import sys; print(sys.prefix)") \
-D WITH_TBB=OFF \
-D WITH_IPP=OFF \
-D WITH_1394=OFF \
-D BUILD_WITH_DEBUG_INFO=OFF \
-D BUILD_DOCS=OFF \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D BUILD_EXAMPLES=OFF \
-D BUILD_TESTS=OFF \
-D BUILD_PERF_TESTS=OFF \
-D WITH_QT=OFF \
-D WITH_GTK=ON \
-D WITH_OPENGL=ON \
-D WITH_GSTREAMER=OFF \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-4.2.0/modules \
-D WITH_V4L=ON  \
-D WITH_FFMPEG=ON \
-D WITH_XINE=ON \
-D BUILD_NEW_PYTHON_SUPPORT=ON \
-D BUILD_opencv_python3=yes \
-D OPENCV_GENERATE_PKGCONFIG=YES \
-D PYTHON3_EXECUTABLE=$(which python) \
-D PYTHON3_INCLUDE_DIR=$(python -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())") \
-D PYTHON3_PACKAGES_PATH=$(python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())") \
-D PYTHON3_NUMPY_INCLUDE_DIRS=$(python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())")"/numpy/core/include"  \
..
~~~

- OPENCV_GENERATE_PKGCONFIG: pkg-config에 등록을 자동으로 해준다.
- 마지막 4줄은 현재 conda의 python3 위치를 찾기위한 부분이다.
- 결과가 나올때 마지막 부분에 python3도 빌드가 되었다고 나와야 정상적으로 빌드된 것이고 마지막 3줄 빌드 완료 메시지가 뜬다.

~~~
--   Python 2:
--     Interpreter:                 /usr/bin/python2.7 (ver 2.7.12)
--     Libraries:                   /usr/lib/x86_64-linux-gnu/libpython2.7.so (ver 2.7.12)
--     numpy:                       /usr/lib/python2.7/dist-packages/numpy/core/include (ver 1.11.0)
--     install path:                lib/python2.7/dist-packages/cv2/python-2.7
--
--   Python 3:
--     Interpreter:                 /home/petopia-01/anaconda3/envs/dog-detect/bin/python (ver 3.7.7)
--     Libraries:                   /home/petopia-01/anaconda3/envs/dog-detect/lib/libpython3.7m.so (ver 3.7.7)
--     numpy:                       /home/petopia-01/.local/lib/python3.7/site-packages/numpy/core/include (ver 1.18.2)
--     install path:                /home/petopia-01/anaconda3/envs/dog-detect/lib/python3.7/site-packages/cv2/python-3.7
--
--   Python (for build):            /usr/bin/python2.7
--
--   Java:
--     ant:                         NO
--     JNI:                         NO
--     Java wrappers:               NO
--     Java tests:                  NO
--
--   Install to:                    /home/petopia-01/anaconda3/envs/dog-detect
-- -----------------------------------------------------------------
--
-- Configuring done
-- Generating done
-- Build files have been written to: /home/petopia-01/opencv/opencv-3.4.10/build
~~~

##### 8. 컴파일 시작

- 아래 명령을 통해 cpu 코어수를 확인한다.

~~~sh
cat /proc/cpuinfo | grep processor | wc -l
~~~

- make명령으로 컴파일을 시작한다. 구동머신 스펙에 따라 다르지만 시간이 꽤 걸린다.
- 마지막 16이라는 숫자는 위 명령을 통해 출력된 cpu 코어수를 적어준 것이다.
- time을 사용하여 진행시간이 얼마나 걸렸는지 확인할 수 있다.

~~~sh
time make -j16
~~~

##### 9. 컴파일된 결과물 설치

~~~sh
# 컴파일된 결과물 설치
sudo make install
~~~


##### 10. 설치 확인

~~~sh
cat /etc/ld.so.conf.d/*
~~~

- 아래처럼 나와야 한다.

~~~
/usr/local/cuda-10.2/targets/x86_64-linux/lib
/usr/lib/x86_64-linux-gnu/libfakeroot
# libc default configuration
/usr/local/lib
# Multiarch support
/lib/x86_64-linux-gnu
/usr/lib/x86_64-linux-gnu
/usr/lib/nvidia-440
/usr/lib32/nvidia-440
/usr/lib/nvidia-440
/usr/lib32/nvidia-440
# Legacy biarch compatibility support
/lib32
/usr/lib32
~~~

- `/usr/local/lib` 부분이 출력되는지 확인한다.
- 나오지 않으면 아래 두 줄을 실행한다.

~~~sh
sudo sh -c 'echo '/usr/local/lib' > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
~~~



##### 참고한 사이트

- [https://webnautes.tistory.com/1030](https://webnautes.tistory.com/1030)
- [https://webnautes.tistory.com/1186](https://webnautes.tistory.com/1186)
- [https://www.nerdluecht.de/index.php/2018/10/15/compile-opencv-3-4-3-for-anaconda-python-3-7](https://www.nerdluecht.de/index.php/2018/10/15/compile-opencv-3-4-3-for-anaconda-python-3-7)
- [https://babel-tower.tistory.com/90](https://babel-tower.tistory.com/90)
