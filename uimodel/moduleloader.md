# 모듈 로더

Fox UI Model에서 모듈 로더는 뷰를 화면에 표시하기 위해 [메뉴 데이터](menudata.md#FoxMenuItem)로부터 뷰 객체가 존재하는 어셈블리(DLL)를 로드하고 객체를 생성하는 역할을 담당합니다. **일반적으로 모듈 로더가 개발자 코드에서 직접 호출되는 경우는 거의 없습니다.** [FoxMenuViewModel](menuview.md#FoxMenuViewModel)은 필요에 따라 모듈 로더를 호출하여 어셈블리를 로드하고 뷰 객체를 생성합니다. `FoxMenuViewModel`이 사용하는 모듈 로더는 `ModuleLoader` 속성을 통해 제공되는 `FoxModuleLoader` 클래스, 혹은 그 파생 클래스 입니다.

목차

* [FoxModuleLoader](#FoxModuleLoader)
* [어셈블리 경로](#어셈블리-경로)
* [모듈 로더와 어플리케이션 배포 방식](#모듈-로더와-어플리케이션-배포-방식)

## FoxModuleLoader

`FoxModuleLoader` 객체는 `FoxMenuViewModel` 객체가 초기화 될 때 생성되어 `ModuleLoader` 속성에 기록됩니다. `FoxMenuViewModel` 객체의 [SelectMenu 메서드가 호출되어 뷰 객체를 생성해야 한다면](menuview.md#메뉴-Action---Open), `FoxMenuViewModel`은 [FoxViewModel](view.md#FoxViewModel) 객체를 생성하고 `FoxModuleLoader` 클래스의 유일한 public 메서드인 `CreateView` 메서드를 호출합니다.

`CreateView` 메서드는 `FoxViewModel` 객체의 `MenuItem` 속성으로부터 [FoxMenuItem](menudata.md#FoxMenuItem) 객체를 얻어내고 이 객체의 `Url` 속성을 이용하여 어셈블리를 로드합니다. 이때 `FoxModuleLoader`는 `Url` 속성 값에 따라 인터넷에서 DLL을 다운로드 할 수도 있습니다. 또, `FoxModuleLoader`는 `FoxMenuItem` 객체의 `ClassName` 속성을 사용하여 로드한 DLL로부터 뷰 객체를 생성합니다. 그리고 이 뷰 객체가 `IFoxView` 인터페이스를 구현하고 있는지도 확인 합니다. 만약 생성한 뷰 객체가 `IFoxView` 인터페이스를 구현하고 있지 않다면 `InvalidCastException` 예외를 유발합니다.

이렇게 뷰 객체가 생성되면 `ViewCreated` 이벤트를 발생합니다. 이 이벤트에 대한 핸들러는 `FoxMenuViewModel`가 구현하고 있습니다 이 이벤트 핸들러에서는 생성된 뷰의 `IFoxView.SecurityContext` 속성을 사용하여 현재 사용자가 이 뷰를 열 수 있는 권한이 있는지 확인합니다. 권한이 있는 경우, `DoViewOpen` 이벤트를 발생하여 생성된 뷰가 [메뉴 뷰](menuview.md)에 의해 UI 적으로 표시하도록 합니다.

이 과정은 개발자가 관여할 필요가 없으며 명시적으로 `FoxModuleLoader`를 호출할 필요 조차 없습니다. 개발자는 단순히 `FoxMenuViewModel` 클래스의 `SelectMenu`를 호출하기만 하면 DLL을 로드하고 뷰 객체를 생성하는 작업이 수행됩니다.

## 어셈블리 경로

`FoxModuleLoader` 객체가 어셈블리를 로드할 때 사용하는 `FoxMenuItem` 객체의 `Url` 속성은 로컬 파일 시스템 경로, UNC 경로, HTTP 경로 등이 모두 사용될 수 있습니다. 또한 `Url` 속성은 상대 경로 혹은 절대 경로도 사용될 수 있습니다. `FoxModuleLoader` 객체는 `Url` 속성 값을 절대 경로로 바꾸기 때문에 상대 경로가 사용된 경우, 몇 가지 규칙에 의해 절대 경로로 변환된 후 어셈블리를 로드 합니다.

ClickOnce로 어플리케이션이 배포된 경우, `Url` 속성에 사용된 **상대 경로**는 ClickOnce 배포의 업데이트 경로(`UpdateLocation` 속성)로부터의 상대 경로가 됩니다. 만약 ClickOnce 어플리케이션이 `http://app.compay.com/deploy` 에서 배포 되었고 `Url` 속성 값이 `Module.dll` 이라면 `FoxModuleLoader` 객체는 `http://app.compay.com/deploy/Module.dll`을 다운로드 받고 이 어셈블리를 로드 하게 됩니다.

어플리케이션이 ClickOnce로 배포되지 않은 경우, `Url` 속성에 사용된 상대 경로는 현재 어플리케이션 도메인(AppDomin)의 베이스 디렉터리로부터의 상대 경로 입니다. 대부분의 데스크톱 어플리케이션들(WinForm, WPF)의 EXE 파일이 존재하는 디렉터리가 어플리케이션 도메인의 베이스 디렉터리가 됩니다. 예를 들어, 어플리케이션이 `C:\company\app.exe` 에 의해 구동되었다면 어플리케이션 도메인의 베이스 디렉터리는 `C:\company`가 되며 `Url` 속성 값이 `modules\module.dll` 이라면 `FoxModuleLoader` 객체는 `C:\company\modules\module.dll` 을 로드하게 됩니다.

따라서, 메뉴 정보의 `Url` 속성 값은 특수한 경우가 아니라면 상대경로 값으로만 사용하는 것이 좋습니다. 상대 경로를 갖는 `Url` 속성 값은 어플리케이션의 배포 시에나 개발 시에 모두 편리하게 이용될 수 있습니다.

## 모듈 로더와 어플리케이션 배포 방식

`FoxModuleLoader`가 뷰를 포함하는 어셈블리를 로드할 때 사용하는 방식은 어플리케이션 배포에 커다란 유연성을 제공합니다.

### NTD(No-Touch-Deploy) 배포 방식

NeoDEEX에서는 메뉴 뷰를 포함하는 EXE 파일과 부속 DLL들을 ClickOnce를 통해 배포하고 뷰를 포함하는 DLL들을 웹 서버에 올려 두는 배포 방식을 **NTD 배포 방식**이라 부릅니다.

이 방식은 상대적으로 크기가 작고 변경이 자주 발생하지 않는 메뉴 뷰를 ClickOnce 패키지를 통해 배포하고 업무 요청에 따라 자주 변경되는 뷰들을 필요할 때마다 웹 서버로부터 다운로드 받아 유지 보수성을 높일 수 있습니다. 이 경우 `FoxModuleLoader`는 ClickOnce 업데이트 경로로부터 상대 경로를 사용하여 어셈블리들을 다운로드 받습니다.

### Updater 배포 방식

NeoDEEX 4.5 버전부터 제공되는 [Fox Updater](/updater/README.md)를 사용하면 어플리케이션 전체 혹은 일부를 로컬 파일 시스템에 다운로드 한 후에 어플리케이션이 구동되도록 할 수 있습니다. 이 경우 어플리케이션은 ClickOnce 배포가 아닌 로컬 파일 시스템에서 구동되게 됩니다. 따라서 어플리케이션 EXE가 다운로드 된 로컬 파일 시스템 경로로부터 상대 경로에서 어셈블리를 로드 하게 됩니다.

### 개발 환경

개발 환경에서는 Visual Studio에서 어플리케이션이 구동되기 때문에 ClickOnce 배포 상황이 아닙니다. FoxModuleLoader가 메뉴 정보의 내용을 사용하여 개발 중인 어셈블리를 로드하도록 하기 위해서는 빌드 결과 EXE 및 DLL 들이 하나의 디렉터리에 모이도록 솔루션의 프로젝트들의 빌드 출력 디렉터리(Output Directory)를 조정해 주는 것이 좋습니다. 구체적인 방법에 대해서는 [STEP 3 – 뷰 작성 예제](tutorial.md#STEP-3-–-뷰-작성)를 참고 하십시오.

---