# 뷰(View)

Fox UI Model에서 메뉴에 의해 표시되는 개별 화면을 뷰라고 합니다. 뷰는 [IFoxView](#IFoxView) 인터페이스를 구현하는 UI 요소입니다. Windows Forms 기반의 어플리케이션이라면 IFoxView 인터페이스를 구현하는 Form, UserControl 등이 뷰의 대상이 될 수 있으며, WPF 어플리케이션이라면 IFoxView 인터페이스를 구현하는 Window, UserControl, Page 등이 뷰의 대상이 될 수 있습니다.

뷰를 생성하기 위해서 뷰가 어떤 DLL에 존재하며 네임스페이스를 포함하는 클래스 이름이 필요합니다. 이렇게 뷰를 관리하기 위한 메타 정보를 담고 있는 객체가 [FoxViewModel](#FoxViewModel) 입니다.

목차

* [FoxViewModel](#FoxViewModel)
* [IFoxView](#IFoxView)
* [메뉴 네비게이션](#%EB%A9%94%EB%89%B4%20%EB%84%A4%EB%B9%84%EA%B2%8C%EC%9D%B4%EC%85%98)

## FoxViewModel

`FoxViewModel` 클래스는 뷰와 1대 1로 대응되는 뷰에 대한 메타 정보 객체 입니다. [FoxMenuViewModel](menuview.md#FoxMenuViewModel)은 뷰를 생성하기 위해 먼저 `FoxViewModel` 객체를 생성하며 이 객체에 메뉴 정보인 [FoxMenuItem](menudata#FoxMenuItem) 객체와 뷰를 나타내는 [IFoxView](#IFoxView) 인터페이스 객체를 각각 `MenuInfo`, `View` 속성에 기록해 둡니다. `FoxViewModel` 객체를 구하면 메뉴 정보, 뷰 정보에 모두 접근할 수 있습니다.

다음은 FoxViewModel 객체의 주요 속성들 입니다.

* `Id` 속성

    `FoxViewModel`을 구분하는 문자열 아이디 입니다. 기본적으로 메뉴 Id와 동일한 값을 가집니다만 항상 그런 것은 아닙니다. 이 속성은 `FoxMenuViewModel`에 의해 설정됩니다.

* `UserInfo` 속성

    현재 사용자에 대한 정보를 담는 [FoxUserInfoContext](/core/security.md#FoxUserInfoContext) 객체를 반환합니다.

* `SecurityContext` 속성

    현재 사용자가 뷰에 대해 갖는 권한 정보를 나타내는 [FoxSecurityContext](/core/security.md#FoxSecurityContext) 객체를 반환합니다. 이 속성의 값은 `FoxMenuViewModel` 객체에 의해 `IFoxVIew.CreateSecurityContext` 메서드 호출을 통해 얻어진 `FoxSecurityContext` 객체를 사용하여 초기화 됩니다.

* `MenuId` 속성

    뷰가 어떤 메뉴에 의해 생성되었는지를 나타내기 위한 메뉴 Id를 반환합니다.

* `MenuInfo` 속성

    뷰가 어떤 메뉴에 의해 생성되었는지를 나타내는 `FoxMenuItem` 객체 입니다.

* `View` 속성

    뷰를 나타내는 `IFoxView` 인터페이스 객체를 반환합니다.

* `MenuViewModel` 속성

    `FoxViewModel` 객체를 생성하고 뷰를 생성한 `FoxMenuViewModel` 객체를 반환합니다.

* `MenuView` 속성

    [메뉴 뷰](menuview.md)를 나타내는 `IFoxMenuView` 인터페이스를 반환합니다.

* PreviousId/NextId 속성

    추후 설명할 메뉴 네비게이션에서 이동 가능한 다음 뷰 혹은 이전 뷰를 나타내는 뷰 Id를 반환합니다.

`FoxViewModel` 객체는 웹 브라우저가 웹 페이지들을 전/후로 이동하는 것과 비슷하게 메뉴 사이를 이동하는 [메뉴 네비게이션](#%EB%A9%94%EB%89%B4%20%EB%84%A4%EB%B9%84%EA%B2%8C%EC%9D%B4%EC%85%98) 기능을 제공합니다. 메뉴 네비게이션 기능은 `FoxViewModel` 객체가 제공하는 `Navigate`, `GoBack`, `MoveTo`, `ReplaceTo` 메서드를 통해 제공됩니다. 메뉴 네비게이션 기능에 대해서는 추후 별도로 설명하겠습니다.

## IFoxView

`IFoxView` 인터페이스는 Fox UI Model에서 객체가 뷰가 되기 위한 최소한의 조건입니다. 하지만 메뉴 뷰가 화면으로써 뷰를 표시하고 Fox UI Model 이 UI 프레임워크(Windows Forms 이나 WPF 등)와 독립적으로 작동하기 위해 구체적인 클래스 대신 IFoxView 인터페이스를 사용하는 것일 뿐입니다. 따라서 뷰는 IFoxView 인터페이스를 구현하는 UI 객체라고 보는 것이 좋습니다.

`IFoxView` 인터페이스는 UI 프레임워크에 독립적으로 구현된 [FoxMenuViewModel](menuview.md#FoxMenuViewModel)이 뷰에 접근할 때 호출되는 메서드/속성들과 비동기 호출 시 프로그래스(progress) UI 표시 지원을 위한 메서드들로 구성되어 있습니다.

일반적으로 `IFoxView` 인터페이스를 일반 개발자가 직접 구현해야 하는 경우는 없습니다. 이미 `IFoxView` 인터페이스를 구현하고 있는 [FoxForm](/winform/foxform.md) 클래스를 사용하거나 프레임워크 커스터마이징 단계에서 NeoDEEX 전문가가 이 인터페이스를 구현하는 베이스 클래스를 작성하고, 일반 개발자는 이들 베이스 클래스에서 파생된 개별 뷰를 작성하는 것이 일반적입니다.

`IFoxView` 인터페이스는 다음과 같은 속성과 메서드로 구성되어 있습니다.

* `ViewModel` 속성

    뷰와 연결된 `FoxViewModel` 객체를 설정하거나 반환합니다. `FoxMenuViewModel` 객체가 뷰  생성 시 `FoxViewModel` 객체를 생성하고 이 속성의 set 메서드를 호출하여 이 값을 초기화 하도록 합니다.

* `SecurityContext` 속성

    뷰는 현재 사용자가 뷰에 대해 어떤 권한을 갖는지 이 속성을 통해 [FoxSecurityContext](/core/security.md#FoxSecurityContext) 객체를 반환해야 합니다. 일반적으로 뷰는 `FoxViewModel`로부터 메뉴 Id와 현재 사용자 정보를 알아내고 이 정보를 이용하여 서버로부터 권한정보를 조회하여 `FoxSecurityContext` 객체를 생성하게 됩니다. 이렇게 생성된 `FoxSecurityContext` 객체를 캐시하는 속성이 바로 `SecurityContext` 속성 입니다.

* `DoInitialize` 메서드

    뷰를 초기화 하도록 `FoxMenuViewModel`이 뷰 객체 생성 이후 호출합니다. `DoInitialize` 메서드는 뷰가 생성될 때 그리고 메뉴 네비게이션에 의해 뷰 이동이 발생할 때에도 호출됩니다.

* `Lock`/`Unlock` 메서드

    [Fox WinForm](winform/introduction.md)에서 비동기 프로그래스 표시를 위한 참조 카운트 증감을 위한 메서드 입니다. 프레임워크 내부에서 사용하기 위한 메서드이며 IFoxView 인터페이스를 직접 구현하고자 하는 경우, NeoDEEX 기술팀에 문의하십시오.

* `GetDesignTimeSecirytContext` 메서드

    프레임워크 내부에서 사용하기 위한 메서드 입니다. Visual Studio의 디자인 타임에서 속성 창에 권한 정보를 표시하기 위해 사용됩니다. `IFoxView` 인터페이스를 직접 구현하고자 하는 경우, NeoDEEX 기술팀에 문의하십시오.

## 메뉴 네비게이션

Fox UI Model은 한 뷰에서 다른 뷰를 열 거나 이전 뷰로 되돌아 갈 수 있는 기능을 제공합니다. 이러한 기능을 메뉴 네비게이션 기능이라 부릅니다. 메뉴 네비게이션 기능은 브라우저에서 링크를 클릭하여 새로운 페이지를 열거나 이전 페이지로 돌아가는 것과 매우 유사한 기능입니다.

메뉴 네비게이트 기능은 `FoxViewModel` 클래스에서 제공하는 `Navigate`, `GoBack`, `MoveTo`, `ReplaceTo` 메서드를 통해 제공됩니다.

### Navigate 메서드

`Navigate` 메서드는 매개변수로 주어진 메뉴 Id를 가진 뷰를 엽니다(open). 만약 주어진 메뉴 Id가 이미 열려 있다면 해당 뷰를 활성화(activate) 합니다. 새롭게 뷰가 Open되는 경우 `FoxMenuViewModel.DoViewOpen` 이벤트가, 기존 뷰가 활성화되는 경우 `FoxMenuViewModel.DoViewActivate` 이벤트가 발생합니다.

다음 코드는 `Navigate` 메서드를 호출하는 뷰의 예제를 보여주고 있습니다.

```csharp
public partial class SampleForm1 : FoxForm
{
    public SampleForm1()
    {
        InitializeComponent();
    }

    private void btnNavigate_Click(object sender, EventArgs e)
    {
        var args = "argument string #1";
        this.ViewModel.Navigate("1020", args);
    }
}
```

`Navigate` 메서드는 2개의 매개변수를 갖습니다. 첫번째 매개변수는 네비게이트 대상이 되는 뷰의 메뉴 Id이며 두번째 매개변수는 뷰가 열리거나 활성화될 때 전달할 매개변수 객체입니다. `Navigate` 메서드가 호출되면 `FoxMenuViewModel` 객체는 열려 있는 뷰들을 확인합니다. 만약 Navigate 메서드가 열고자 하는 뷰가 이미 열려 있다면 해당 뷰를 활성화(activate) 합니다. 뷰가 열려 있지 않다면 새로운 뷰를 열게 됩니다.

`FoxMenuViewModel`은 뷰가 열려 있는지 여부에 상관없이 `IFoxView.DoInitialize` 메서드를 호출합니다. 이 메서드는 메뉴 선택([SelectMenu](menuview.md#%EB%A9%94%EB%89%B4%20Action%20-%20Open))에 의해 뷰가 열릴 때에도 호출됩니다. 두 상황에서 다른점은 `IFoxView.DoInitialize` 메서드로 전달되는 매개변수입니다. 메뉴 선택에 의해 `DoInitialize` 메서드가 호출되는 경우 매개변수는 항상 null 이지만 메뉴 네비게이트에 의해 `DoInitialize` 메서드가 호출된 경우에는 메뉴 네비게이트 메서드들(`Navigate`, `MoveTo` 등)이 전달한 매개변수가 전달됩니다.

다음은 `IFxView.DoInitialize` 메서드를 구현하는 [FoxForm](/winform/foxform.md)의 `OnViewInitialize` 메서드를 구현하는 예를 보여줍니다. `FoxForm` 클래스는 `IFoxView` 인터페이스를 구현하고 있으며 `IFoxView.DoInitialize` 메서드가 호출되면 `OnViewInitialize` 메서드를 호출합니다.

```csharp
public partial class SampleForm2 : FoxForm
{
    public SampleForm2()
    {
        InitializeComponent();
    }

    protected override void OnViewInitialize(object args)
    {
        label2.Text = $"Argument: {args}";
    }
}
```

위와 같은 구현에서 메뉴 선택에 의해 `SimpleForm2` 가 열리는 경우 `args` 매개변수는 null 이 되며, 위에서 보았던 `Navigate` 메서드 호출에 의해 `SimpleForm2`가 열리거나 활성화 되는 경우 `args` 매개변수의 값은 "argument string #1" 이 됩니다. `Navigate` 메서드에 의해 `OnViewInitialize` 메서드로 전달되는 매개변수는 object 타입으로 문자일 뿐만 아니라 데이터 테이블, 일반 객체 등 다양한 정보를 전달할 수 있습니다.

### CanGoBack 속성/GoBack 메서드

`Navigate` 메서드 호출에 의해 열린 뷰는 이전 뷰, 즉 `Navigate` 메서드를 호출한 뷰를 활성화할 수 있습니다. 이를 위해 `GoBack` 메서드를 호출하면 됩니다. 또, `CanGoBack` 속성을 사용하여 되돌아 갈 뷰가 존재하는지 여부도 파악할 수 있습니다. 다음 코드는 `CanGoBack` 속성과 `GoBack` 메서드를 사용하여 `Navigate` 메서드를 호출한 뷰로 제어를 옮길 수 있습니다.

```csharp
private void button1_Click(object sender, EventArgs e)
{
    if (this.ViewModel.CanGoBack == true)
    {
        this.ViewModel.GoBack();
    }
}
```

`GoBack` 메서드에 의해 뷰가 활성화 되는 경우 `IFoxView.DoInitialize` 메서드는 호출되지 않습니다.

### MoveTo 메서드

메뉴 네비게이션의 `MoveTo` 메서드는 `Navigate` 메서드와 매우 유사 합니다. 주어진 메뉴 Id를 새롭게 열거나 이미 열려 있는 뷰를 활성화하는 작동방식은 `Navigate` 메서드와  동일합니다.

```csharp
private void btnMoveTo_Click(object sender, EventArgs e)
{
    var args = "argument string #2";
    this.ViewModel.MoveTo("1020", args);
}
```

하지만 `MoveTo` 메서드에 의해 열린 뷰에서 차이가 납니다. `Navigate`에 의해 열린 뷰는 `GoBack` 메서드를 호출하여 이전 뷰를 활성화 할 수 있지만 `MoveTo`에 의해 열린 뷰는 `GoBack` 메서드에 의해 이전 뷰를 활성화 할 수 없습니다. 당연하게 `MoveTo`에 의해 열린 뷰에서 `CanGoBack` 속성은 false 를 반환합니다.

### ReplaceTo 메서드

메뉴 네비게이션의 `ReplaceTo` 메서드는 이 메서드를 호출한 뷰를 닫고 주어진 메뉴 Id를 가진 뷰를 엽니다. 만약 해당 뷰가 이미 열려 있다면 뷰를 활성화 합니다.

```csharp
private void btnReplaceTo_Click(object sender, EventArgs e)
{
    var args = "argument string #3";
    this.ViewModel.ReplaceTo("1020", args);
}
```

`ReplacteTo` 메서드 호출이 정상적으로 뷰를 닫기 위해서는 메뉴 뷰는 `DoViewClose` 이벤트 핸들러를 구현해야 합니다. 만약 뷰가 정상적으로 닫히지 않으면 새로운 뷰도 열리지 않습니다. 다음은 `DoViewClose` 이벤트를 구현하는 예제 입니다.

```csharp
_menuViewModel.DoViewClose += CloseView;

private void CloseView(object sender, FoxViewEventArgs e)
{
    var form = (FoxForm)e.ViewModel.View;
    form.Close();
}
```

---