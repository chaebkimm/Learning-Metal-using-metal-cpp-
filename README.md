# Learning-Metal-using-metal-cpp-한글 번역본
<a href="https://developer.apple.com/kr/metal/" title="Apple Developer Homepage">metal c++ 샘플 코드로 Metal을 공부하기</a> 파일을 한글로 번역했습니다.
원문에 없지만 이해하는데 도움이 될 수 있는 추가 정보는 **로 표시했습니다.

## Overview
본 프로젝트에 있는 C++ 예제 코드 시리즈는 Metal API를 소개하고 이를 이용해서 일반적인 그래픽 작업을 하는 방법을 소개합니다.

각 예제는 이전 예제에서 최소한의 코드 변화로 새로운 기능을 도입합니다. diff 툴을 써서 정확히 어떤 코드가 도입된 기능에 필요한지 볼 수 있습니다.

본 README는 Metal을 써서 어떻게 프로젝트를 만들고 각각의 특정 작업을 수행할지를 설명합니다.

프로젝트는 다음과 같은 작업을 수행합니다.

* 00 - Metal 렌더링을 위한 윈도우 창 생성
* 01 - 삼각형 렌더링
* 02 - 셰이더 정보 저장
* 03 - 에니메이션
* 04 - 여러 객체 인스턴스 그리기
* 05 - 3D 관점으로 렌더링하기
* 06 - 조명
* 07 - 텍스쳐
* 08 - GPU 계산
* 09 - GPU 계산 결과 렌더링
* 10 - GPU 디버깅

## Dependencies
본 예제는 metal-cpp, metal-cpp-extensions 라이브러리를 포함하고 있습니다.

예제에 첨부된 Xcode 프로젝트를 쓰거나 UNIX make로 프로젝트를 빌드하면 실행가능합니다.

본 프로젝트는 C++17 이상이 필요합니다 (Xcode 9.3 이상에 있습니다).

## Building with Xcode
Xcode에서 LearnMetalCPP.xcodeproj 프로젝트 열기를 실행합니다. 

프로젝트에는 각 예제의 타겟이 있고, 모든 예제를 한번에 빌드하는 타겟도 있습니다.

실행 버튼 옆에 있는 scheme 드롭다운 메뉴에서 어떤 예제를 빌드할지 선택하고 실행합니다.

## Building with Make
Makefile로 예제를 빌드하기 위해서는, 터미널을 열고 make 명령을 실행합니다. 빌드 프로세스가 실행 파일을 build/ 폴더에 만들 것입니다. 

디폴트 설정으로 Makefile은 소스 코드를 -O2 최적화 레벨에서 컴파일합니다. 다음과 같은 옵션값을 make 명령의 인자에 추가해서 빌드 설정을 바꿀 수 있습니다.
* DEBUG=1 : 최적화를 비활성화시키고 변수나 함수의 이름 정보를 포함시킴 (GCC의 -g 옵션)
* ASAN=1 : 메모리 오류 탐지기인 address sanitizer 지원을 받아서 빌드 (GCC의 -fsanitize=address 옵션)

## 




