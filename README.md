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

## Sample 0: Create a Window for Metal Rendering
00-window 예제는 Metal을 활용해서 그려진 내용을 보여주는 윈도우가 있는 맥OS 어플리케이션을 만드는 방법을 보여줍니다. 본 예제에서는 윈도우의 내용을 모두 지우고 빨간색을 칠합니다.

프로그램은 main 함수로 시작합니다.

```c++
MyAppDelegate del;

NS::Application* pSharedApplication = NS::Application::sharedApplication();
pSharedApplication -> setDelegate( &del );
pSharedApplication -> run();
```

윈도우를 만들기 위해서, 공유 어플리케이션 객체를 생성하고 사용자 지정 어플리케이션 대행자(NS::ApplicationDelegate 서브클래스의 인스턴스)를 설정합니다. 대행자는 시스템 이벤트 알림을 받고, 특별히 어플리케이션 실행시 로딩이 끝나고 윈도우를 만들 준비가 되었다는 알림을 받습니다. 

어플리케이션 로딩이 끝났다는 알림을 받으면 applicationDidFinishLaunching() 메소드가 실행됩니다. 본 예제에서는 이 메소드를 오버라이드하여 윈도우, 메뉴, Metal 내용 뷰를 생성합니다.

본 예제에서는 MTK::View 클래스를 써서 Metal 내용을 화면에 나타냅니다. MTK::View는 또한 일정 시간마다 렌더링을 실행시키는 반복문을 제공합니다. applicationDidFinishLaunching() 메소드는 뷰를 초기화시키며, 초기화 과정에서는 CGRect 객체에 뷰의 가로, 세로 길이를 저장하고, MTL::Device 객체에 시스템 GPU에 접근할 수 있는 루트를 저장합니다. 이 메소드는 또한 뷰의 렌더링 타겟의 픽셀 포맷과 배경색을 설정합니다.

```c++
_pMtkView = MTK::View::alloc()->init( frame, _pDevice );
_pMtkView->setColorPixelFormat( MTL::PixelFormat::PixelFormatBGRA8Unorm_sRGB );
_pMtkView->setClearColor( MTL::ClearColor::Make( 1.0, 0.0, 0.0, 1.0 ) );
```

applicationDidFinishLaunching() 메소드는 또한 MyMTKViewDelegate 클래스 인스턴스를 대행자로 설정합니다.

```c++
_pViewDelegate = new MyMTKViewDelegate( _pDevice );
_pMtkView->setDelegate( _pViewDelegate );
```

MyMTKViewDelegate는 MTK::ViewDelegate 클래스의 서브클래스입니다. MTK::ViewDelegate는 MTK::View가 이벤트를 포워드할 수 있는 인터페이스를 제공합니다. 슈퍼클래스의 가상 함수를 오버라이딩 함으로써 MyMTKViewDelegate는 해당 이벤트에 반응할 수 있습니다. MTK::View는 drawInMTKView() 메소드를 프레임마다 호출해서 렌더링을 업데이트합니다.

```c++
void MyMTKViewDelegate::drawInMTKView( MTK::View* pView )
{
    _pRenderer->draw( pView );
}
```

drawInMTKView()는 Renderer 클래스의 draw() 메소드를 호출합니다. draw() 메소드는 뷰의 배경색을 설정하는데 필요한 최소한의 작업을 합니다.

```c++
MTL::CommandBuffer* pCmd = _pCommandQueue->commandBuffer();
MTL::RenderPassDescriptor* pRpd = pView->currentRenderPassDescriptor();
MTL::RenderCommandEncoder* pEnc = pCmd->renderCommandEncoder( pRpd );
pEnc->endEncoding();
pCmd->presentDrawable( pView->currentDrawable() );
pCmd->commit();
```

draw()메소드는 다음과 같은 일을 합니다:
1. 명령 버퍼 객체를 만듭니다. 이를 통해 GPU에서 실행할 명령들을 인코딩할 수 있습니다.
2. 렌더링 명령어 인코더 객체를 만듭니다. 명령어 인코더는 명령 버퍼가 그리기 명령어를 받을 수 있게 준비시키고, 그리기 시작과 끝에서 실행할 작업을 지정합니다. 
3. current drawable을 화면에 나타내 GPU 작업 결과를 화면에 보이게 하는 명령을 인코딩합니다.
4. 명령 버퍼를 GPU에서 실행시키기 위해서 명령 큐에 제출합니다. 

본 예제에서 MTLRenderCommandEncoder 객체는 명시적으로 어떤 명령도 인코딩하지 않습니다. 하지만, 인코더를 생성할 때 쓰인 MTL::RenderPassDescriptor 객체가 화면 비우기 명령을 인코딩합니다. 그 결과 뷰가 빨간색으로 칠해지게 됩니다.

> [!NOTE]
> Metal은 자동해제된 임시 객체에 의존합니다. 본 예제는 프레임이 시작될 때 NS::AutoreleasePool 객체를 만들어서 자동해제된 임시 객체들을 관리합니다. NS::AutoreleasePool은 임시 객체들을 트래킹하고, 프레임이 끝나고 NS::AutoreleasePool의 소멸자가 호출되었을 때 임시 객체들을 해제합니다. 자세한 내용은 <a href="https://developer.apple.com/documentation/foundation/nsautoreleasepool/" title="NSAutoreleasePool | Apple Developer Documentation">metal 문서</a>에 있습니다.


