# Fox UI Model

사용자가 응용 프로그램을 사용하기 위해서는 일련의 메뉴 체계가 필요하고 이 메뉴 체계로부터 어떤 메뉴를 선택했을 때 메뉴에 해당하는 화면이 구동 되어야 합니다. 따라서 응용 프로그램은 어떤 메뉴들이 어떠한 구조로 구성되어 있는지를 관리할 수 있어야 하며, 이들 메뉴를 다양한 형태(풀다운 메뉴, 트리 메뉴, 리본 메뉴 등)로 표시할 수 있어야 하고, 메뉴가 선택되었을 때 해당 메뉴를 나타내는 화면을 표시할 수 있도록 체계를 갖추어야 합니다.

NeoDEEX에서 제공하는 **Fox UI Model은 클라이언트가 메뉴들을 관리하고 표시하며, 메뉴가 선택되었을 때 화면을 로드하고 표시할 수 있는 프레임워크를 제공**합니다. 메뉴를 로드하고 계층적으로 관리할 수 있는 `FoxMenuManager` 클래스, 메뉴와 대응되는 화면을 담고 있는 DLL을 로드하고 화면의 인스턴스를 생성할 수 있는 `FoxModuleLoader` 클래스, 화면을 나타내는 `IFoxView` 인터페이스, 그리고 이들을 서로 유기적으로 연결해 주는 `FoxMenuView` 클래스를 이용하면 빠르게 계층적인 메뉴 데이터를 구축하고 메뉴를 표시하는 메인 어플리케이션과 개별 화면을 손쉽게 연결 할 수 있습니다.

이 문서는 다음과 같은 항목들로 구성되어 있습니다.

* [Fox UI Model 개요](introduction.md)

* [메뉴 작성 튜토리얼](tutorial.md)

* [메뉴 데이터 구성](menudata.md)

* [메뉴 뷰](menuview.md)

* [모듈 로더](moduleloader.md)

* [뷰(View)](view.md)

## Summary

NeoDEEX는 Fox UI Model을 통해 WinForm, WPF 등 UI 프레임워크에 독립적으로 메뉴 데이터를 읽어 계층 구조로 구성하고 사용자의 메뉴 선택에 따라 화면(뷰)을 담고 있는 어셈블리(DLL)을 로드하고 화면 객체를 생성하여 표시하는데 필요한 코드 뼈대를 제공합니다.

Fox UI Model 에서 메뉴와 뷰를 제어하는데 핵심적인 역할을 하는 클래스는 [FoxMenuViewModel](menuview.md#FoxMenuViewModel) 입니다. FoxMenuViewModel은 FoxMenuManager 클래스를 통해 메뉴를 읽어 FoxMenuItem 객체의 계층적인 데이터 구조를 구성하고 메뉴를 표시하기 위한 이벤트(DoMenuItemCreate)를 발생합니다. FoxMenuViewModel 클래스는 사용자가 메뉴를 선택할 때 호출할 수 있는 SelectMenu 메서드를 제공하여 FoxModuleLoader 클래스를 호출하여 어셈블리를 로드하고 뷰를 생성합니다. 그리고 DoViewOpen 이벤트를 발생하여 메뉴 뷰로 하여금 뷰가 나타나도록 지시합니다. 이미 뷰가 열려 있다면 DoViewActivate 이벤트를 발생하여 메뉴 뷰가 열려 있는 뷰가 활성화 되도록 할 수도 있습니다.

메뉴가 선택됨에 따라 나타나는 뷰는 그 정보를 포함하는 FoxViewModel 객체와 1대 1로 대응되어 관리됩니다. FoxViewModel 객체는 뷰에 연결된 메뉴 정보, 권한 정보등을 포함하고 있으며 실제로 FoxMenuViewModel 클래스는 FoxViewModel 객체를 통해 뷰를 제어합니다. FoxViewModel 클래스는 뷰에 대한 정보 뿐만 아니라 메뉴들 사이를 네비게이트 할 수 있는 다양한 메서드들도 제공합니다.

개발자는 FoxMeuViewModel, FoxViewModel 클래스와 IFoxView 인터페이스를 통해 메뉴, 뷰의 연동을 손쉽게 관리할 수 있으며 이를 통해 강력하고 충분한 UX를 가진 메뉴 시스템을 구축할 수 있습니다.
