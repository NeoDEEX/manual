# 메뉴 데이터 구성

Fox UI Model의 핵심은 XML 혹은 데이터베이스로부터 메뉴 정보를 읽어 메뉴 데이터 구조를 구성하고 메뉴 화면에 표시하는 것입니다. 메뉴 정보로부터 메뉴 데이터 구조를 구성하는데 사용되는 핵심 클래스는 `FoxMenuManager` 클래스 입니다.

목차

* [FoxMenuManager 클래스](#FoxMenuManager)
* [FoxMenuItem 클래스](#FoxMenuItem)
* [메뉴 객체 모델](#%EB%A9%94%EB%89%B4%20%EA%B0%9D%EC%B2%B4%20%EB%AA%A8%EB%8D%B8)
* [메뉴 XML](#%EB%A9%94%EB%89%B4%20XML)
* [메뉴 확장](#%EB%A9%94%EB%89%B4%20%ED%99%95%EC%9E%A5)

## FoxMenuManager

`FoxMenuManager` 클래스는 XML 혹은 데이터베이스로부터 메뉴를 로드하여 계층적인 구조의 메뉴 구조를 구성합니다. 각 메뉴 항목은 [FoxMenuItem](#FoxMenuItem) 클래스로 표현되며 `FoxMenuItem`들은 계층적인 구조로 구성되어 `FoxMenuManager` 내부에 저장됩니다.

`FoxMenuManger` 클래스의 인스턴스는 [FoxMenuViewModel](menuview.md#FoxMenuViewModel) 객체가 기본적으로 생성하며 MenuManager 속성을 통해 접근할 수 있습니다.

### 메뉴 로드

메뉴 정보를 로드하기 위해서는 `FoxMenuManager` 클래스가 제공하는 `LoadMenu` 메서드를 사용합니다. 대표적인 `LoadMenu` 메서드는 메뉴 XML 파일의 경로를 매개변수로 하는 메서드들 입니다. `Uri` 객체 혹은 문자열을 통해 메뉴 XML 파일의 경로가 주어지면 `LoadMenu` 메서드는 해당 XML 파일을 읽어 `FoxMenuItem` 객체들을 구성하여 XML과 동일한 계층 구조를 구성하게 됩니다. 다음 코드는 메뉴를 로드하는 예를 보여줍니다.

```csharp
FoxMenuManager manager = new FoxMenuManager();
manager.LoadMenu("./UIModel/Menu.xml");
```

> 이 예제에서는 `FoxMenuManager` 인스턴스를 직접 생성하였지만 대개의 경우, `FoxMenuViewModel` 객체에서 제공하는 `MenuManager` 속성을 사용하는 것이 일반적입니다.

`LoadMenu` 메서드에 사용하는 메뉴 XML 파일의 경로는 URL 혹은 로컬 파일 경로가 사용될 수 있습니다. URL이 사용되면 `LoadMenu` 메서드는 주어진 URL로부터 XML을 다운로드 받습니다. 로컬 파일 경로의 경우, 절대 경로 혹은 상대 경로가 사용될 수 있으며 상대 경로가 사용되면 현재 `AppDomain`의 베이스 디렉터리 로부터의 상대 경로로 간주 됩니다.

> 대개의 경우, `AppDomain`의 베이스 디렉터리는 대개 exe 파일과 같은 디렉터리 입니다. 하지만 ASP.NET 웹 어플리케이션의 경우 웹 어플리케이션의 루트 디렉터리가 `AppDomain`의 베이스 디렉터리입니다. `AppDomain`이 코드에 의해 생성되는 경우, 명시적으로 설정될 수도 있습니다.

메뉴 XML에 대한 경로는 `LoadMenu` 메서드의 매개변수로 주어질 수도 있지만 `Location` 속성을 통해 주어질 수도 있습니다. `Location` 속성에 메뉴 XML에 대한 경로를 설정하고 매개변수 없이 `LoadMenu` 메서드를 호출하면, `Location` 속성이 나타내는 메뉴 XML을 로드하게 됩니다.

```csharp
FoxMenuManager manager = new FoxMenuManager();
menuManager.Location = "./UIModel/Menu.xml";
menuManager.LoadMenu();
```

`FoxMenuManager` 클래스는 XML 파일 뿐만 아니라 문자열 혹은 `Stream`으로부터 메뉴 XML을 로드할 수도 있습니다. 메뉴 XML이 문자열로서 존재하는 경우 `LoadXml` 메서드를 호출하면 됩니다. 파일이 아닌 다른 저장소(리소스, 네트워크 서버 등)에 존재하는 메뉴 XML을 로드해야 하는 경우 `Stream`을 매개변수로 사용하는 `LoadMenu` 메서드를 사용하면 됩니다.

많은 경우, 메뉴 정보는 데이터베이스에 기록되어 있기 마련입니다. 데이터베이스에 존재하는 메뉴 정보를 사용해야 하는 경우, 데이터베이스로부터 메뉴 정보를 조회하고 이 정보로부터 개별 `FoxMenuItem` 객체들을 구성해야 합니다. 다음은 데이터베이스 테이블에 기록된 메뉴 정보를 읽어 `FoxMenuItem` 객체들을 구성하고 `LoadMenu` 메서드를 호출하는 예제를 보여주고 있습니다.

```csharp
var dt = GetMenuDataFromDatabase();
var list = new List<FoxMenuItem>();
foreach(DataRow row in dt.Rows)
{
    var menuInfo = new FoxMenuItem();
    menuInfo.Id = (string)row["ID"];
    menuInfo.Title = (string)row["TITLE"];
    menuInfo.ParentId = (string)row["PARENT_ID"];
    menuInfo.Enabled = (string)row["USE_YN"] == "Y";
    menuInfo.Url = (string)row["DLL"];
    menuInfo.ClassName = (string)row["CLASS"];
    menuInfo.MenuAction = (FoxMenuAction)((int)row["ACTION"]);
    list.Add(menuInfo);
}

var menuManager = new FoxMenuManager();
menuManager.LoadMenu(list);
```

`FoxMenuItem` 컬렉션으로부터 메뉴를 로드할 때 `Id` 속성과 `ParentId` 속성에 의해 계층적인 메뉴 구조가 구성됨에 유의하십시오.

### 메뉴 저장

`FoxMenuManager`에 저장된 메뉴 정보들은 다시 XML 형태로 저장될 수 있습니다. `SaveMenu` 메서드는 `Location` 속성이 지정하는 경로에 메뉴 XML을 저장합니다. 메뉴 XML의 경로는 메뉴 로드와 마찬가지로 URL 혹은 로컬 파일 경로가 사용될 수 있습니다. URL이 사용되면 메뉴 XML은 POST 메서드를 사용하여 주어진 URL에 HTTP 요청 메시지를 전송합니다. 로컬 파일 경로의 경우, 해당 파일에 메뉴 XML을 기록합니다.

`SaveMenu` 메서드 외에도 메뉴 XML을 저장하는 다양한 방식을 지원하기 위해 추가적인 메서드들을 제공합니다. `GetMenuXml` 메서드는 메뉴 XML을 문자열로 반환하며 `WriteMenuXml` 메서드는 `Stream` 객체에 메뉴 XML을 기록합니다. 이들 메서드를 사용하여 다양한 매체에 메뉴 XML을 저장할 수 있습니다. 이들 두 메서드는 수정된 내 메뉴(My Menu)를 기록하는데 주로 사용됩니다.

## FoxMenuItem

`FoxMenuItem` 클래스는 각 메뉴 정보를 나타내는 클래스 입니다. `FoxMenuManager`는 `FoxMenuItem` 객체들을 계층적 구조로 보관하며 [FoxMenuViewModel](menuview.md#FoxMenuViewModel) 객체는 선택된 메뉴에 대해 어떤 행동을 수행할지 역시 `FoxMenuItem` 객체에 기록된 정보를 사용합니다. 또한 [FoxMenuLoader](moduleloader.md#FoxMenuLoader)는 `FoxMenuItem` 객체에 기록된 DLL의 경로와 클래스 이름을 사용하여 DLL을 다운로드하고 화면 객체(`Form`, `UserControl` 등)를 생성합니다.

`FoxMenuItem` 클래스의 핵심적인 속성들은 다음과 같습니다.

* `Id` 속성

    메뉴 아이템을 구별하는 ID 입니다. `FoxMenuManager` 객체 내부에서 중복되지 않는 유일한 값이어야 합니다.

* `Title` 속성

    메뉴 아이템에 대한 이름을 나타냅니다. 대부분의 경우, `Title` 속성의 값이 UI 로 표시됩니다.

* `MenuAction` 속성

    `FoxMenuAction` 열거형으로써 메뉴 아이템이 선택되었을 때 수행해야 하는 작업을 나타냅니다. `Auto` 값은 `FoxMenuViewModel`에 의해 자동으로 수행할 작업이 선택됩니다. 메뉴 아이템이 자식 아이템을 가진 경우 Expand 작업으로 간주하여 `FoxMenuViewModel`은 `DoMenuItemExpand` 이벤트를 발생시킵니다. 메뉴 아이템이 자식 아이템을 가지고 있지 않은 경우 `Open` 작업으로 간주하여 `DoViewOpen` 이벤트를 발생시킵니다.

    `Custom` 값이 명시적으로 지정된 경우, `DoMenuItemCustom` 이벤트를 발생시킵니다. 이 이벤트에서 어플리케이션에서 정의하는 임의의 작업(웹 페이지 열기 등)을 수행할 수 있습니다.

* `Url` 속성

    메뉴 아이템이 지시하는 메뉴를 획득할 수 있는 URL을 나타냅니다. http로 시작하는 인터넷 URL 혹은 로컬 파일에 대한 절대/상대 경로를 사용할 수 있습니다. `MenuAction` 속성에 의해 `Open` 작업이 수행되는 경우 `FoxMenuLoader` 클래스는 이 속성의 값을 사용하여 어셈블리(DLL)을 로드 하게 됩니다. `MenuAction` 속성이 `Open` 작업이 아닌 경우, 이 속성값은 무시됩니다. `Custom` 작업에서는 `DoMenuItemCustom` 이벤트 내에서 이 속성값을 임의의 용도로 해석하여 사용할 수 있습니다.

* `ClassName` 속성

    메뉴 아이템이 지시하는 뷰에 대한 클래스 이름입니다. 클래스 이름은 네임스페이스를 포함하는 전체 이름이며 `MenuAction` 속성에 의해 `Open` 작업이 수행되는 경우 `FoxMenuLoader` 클래스는 이 속성의 값을 사용하여 뷰 객체를 생성합니다. `MenuAction` 속성이 `Open` 작업이 아닌 경우, 이 속성값은 무시됩니다. `Custom` 작업에서는 `DoMenuItemCustom` 이벤트 내에서 이 속성값을 임의의 용도로 해석하여 사용할 수 있습니다.

* `ParentId` 속성

    메뉴의 부모의 메뉴 Id를 나타냅니다. `FoxMenuManager`는 메뉴를 로드할 때 `FoxMenuItem` 객체들의 선형 컬렉션을 구성하고 `Id` 속성과 `ParentId` 속성을 사용하여 계층 구조를 만듭니다. 따라서 XML과 같이 자체적인 계층 구조를 갖지 않는 저장소(데이터베이스 등)에서 `FoxMenuItem` 객체를 생성해 낼 때 이 속성의 값을 설정하는 것이 매우 중요합니다.

* `Visible`, `Enabled`, `Expanded` 속성

    메뉴가 펼쳐져야 하거나, 숨김 혹은 비활성화 되어야 함을 나타냅니다. 이 속성들의 값은 메뉴의 초기 상태를 나타냅니다. 메뉴가 UI로 표현된 이후 펼쳐짐 여부, 숨김, 활성화 여부를 지속적으로 반영하지 않음에 유의해야 합니다.

* `DisplayTitle` 속성

    `Title` 속성과 다른 이름으로 메뉴를 표시하고자 하는 경우 `DisplayTitle` 속성을 명시하면 됩니다. 실제로 Fox UI Model은 기본적인 메뉴의 이름으로 `DisplayTitle` 속성을 사용합니다. `DisplayTitle` 속성이 명시적으로 지정되지 않으면 `Title` 속성의 값이 `DisplayTitle` 속성의 값으로 사용됩니다.

* `ExtraInfo` 속성

    문자열 타입의 `ExtraInfo` 속성은 임의의 정보를 기록하여 어플리케이션에서 임의로 사용할 수 있습니다. 예를 들어 메뉴에 대한 추가적인 권한 정보를 이 속성에 기록하고 메뉴가 선택될 때 현재 사용자가 `ExtraInfo`에 기록된 권한 정보를 만족하는지 확인하여 메뉴를 열기를 수행할 것인지 판단할 수 있습니다.

* `Parent`, `MenuItems` 속성

    `FoxMenuManager`에 의해 메뉴가 로드 된 이후에 설정되는 속성들로써 부모 메뉴와 자식 메뉴(들)를 나타냅니다.

* `IsRoot`, `IsLeaf`, `HasItems`, `HasViewModel` 속성

    메뉴 아이템이 최상위 아이템인지, 최하위 아이템인지, 자식 메뉴들을 가지고 있는지, DLL/Class 이름이 지정되어 있는지를 나타내는 플래그 속성 입니다.

## 메뉴 객체 모델

`FoxMenuManager` 클래스와 `FoxMenuItem` 클래스는 객체 모델을 구성하여 계층적인 메뉴를 표시합니다. 메뉴 계층 구조에서 최상위 메뉴들, 즉 부모를 갖지 않는 루트 메뉴들은 `FoxMenuManager` 클래스의 `RootItems` 속성에서 접근할 수 있습니다. `RootItems` 속성은 `FoxMenuItemCollection` 클래스 타입으로 최상위 메뉴들에 대한 컬렉션을 제공합니다. 최상위 메뉴로부터 각 메뉴의 하위 메뉴는 `FoxMenuItem` 클래스의 `MenuItems` 속성을 통해 접근할 수 있습니다.

다음 코드는 반복문과 재귀 호출(recursive call)을 사용하여 트리 뷰 컨트롤에 메뉴를 표시하는 예를 보여줍니다. 앞서 메뉴 작성 튜토리얼의 [STEP 2 - 메뉴 로드 및 메뉴 표시](tutorial.md#STEP%202%20-%20%EB%A9%94%EB%89%B4%20%EB%A1%9C%EB%93%9C%20%EB%B0%8F%20%EB%A9%94%EB%89%B4%20%ED%91%9C%EC%8B%9C)에서 풀 다운 메뉴를 통해 메뉴를 표시하는 방법과 매우 유사합니다.

```csharp
private void CreateMenuItem(object sender, FoxMenuItemEventArgs e)
{
    var menuInfo = e.MenuInfo;
    var node = treeView1.Nodes.Add(menuInfo.DisplayTitle);
    node.Tag = menuInfo;
    node.ImageKey = "folder";
    AddChildTreeNode(node, menuInfo);
}

private void AddChildTreeNode(TreeNode parentNode, FoxMenuItem parentMenuInfo)
{
    foreach(var menuInfo in parentMenuInfo.MenuItems)
    {
        var node = parentNode.Nodes.Add(menuInfo.DisplayTitle);
        node.Tag = menuInfo;
        if (menuInfo.HasItems == true)
        {
            node.ImageKey = "folder";
            AddChildTreeNode(node, menuInfo);
        }
        else
        {
            node.ImageKey = "app";
        }
    }
}
```

## 메뉴 XML

메뉴 XML은 [FoxMenuManager](#FoxMenuManager) 클래스가 메뉴 정보를 읽어 들이는 가장 기본적인 수단입니다. 메뉴 XML은 루트 요소(elment)인 `<menuTree>` 요소와 `<menuItem>` 요소들의 계층 구조로 매우 간단한 구조를 이룹니다. 다음은 간단한 메뉴 XML의 예를 보여주고 있습니다.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<menuTree>
  <menuItem id="10" title="화면예제">
    <menuItem id="1010" title="화면예제1" url="UIModules.dll" class="UIModules.SampleForm1" />
    <menuItem id="1020" title="화면예제2" url="UIModules.dll" class="UIModules.SampleForm2" />
    <menuItem id="1030" title="하위그룹">
      <menuItem id="1031" title="화면예제3" url="UIModules.dll" class="UIModules.SampleForm3" />
      <menuItem id="1032" title="화면예제4" url="UIModules.dll" class="UIModules.SampleForm4" />
    </menuItem>
  </menuItem>
  <menuItem id="20" title="권한예제" />
  <menuItem id="30" title="인사관리" />
  <menuItem id="40" title="시스템관리"/>
</menuTree>
```

### `<menuTree>` 요소

`<menuTree>` 요소는 메뉴 XML의 루트 요소로만 사용되며 특별한 속성(attribute)을 갖지 않는 XML 요소 입니다.

### `<menuItem>` 요소

`<menuItem>` 요소는 각 메뉴 아이템에 대한 모든 정보를 포함하는 메뉴 XML의 핵심적인 XML 요소입니다. `<menuItem>` 요소는 하위에 다시 `<menuItem>` 요소를 가질 수 있으며 이러한 계층 구조는 메뉴 계층 구조에 그대로 반영됩니다.

`<menuItem>` 요소는 `FoxMenuItem` 객체와 1대 1로 대응되며 `<menuItem>` 요소에 기록된 모든 정보는 `FoxMenuItem` 객체를 통해 접근이 가능합니다. 다음 표는 `<menuItem>` 요소의 속성과 `FoxMenuItem` 클래스의 속성이 어떻게 대응되는지 보여 줍니다.

|`<menuItem>` 속성|`FoxMenuItem` 속성|필수여부|예제|
|-----|-----|-----|-----|
|id|Id|예|P101|
|title|Title|예|예제 메뉴|
|action|Action|아니오|Auto/Open/Expand/Custom|
|url|Url|예/아니오|`http://sever/Module.dll`|
|class|ClassName|예/아니오|MyNamespace.MyClass|
|visible|Visible|아니오|true/false|
|enabled|Enabled|아니오|true/false|
|expanded|Expanded|아니오|true/false|
|displayTitle|DisplayTitle|아니오|예제 메뉴 #1|
|extraInfo|ExtraInfo|아니오|Role=Admin;|

`<menuItem>` 요소에 나타나지 않는 `FoxMenuItem`의 속성들은 `ParentId`, `Parent`, `IsLeaf` 등의 속성으로써 메뉴 정보가 `FoxMenuManager`에 로드됨에 따라 메뉴 계층 구조 등에 의해 자동으로 설정되기 때문에 메뉴 XML에 기록될 필요가 없는 정보입니다.

메뉴 XML로부터 메뉴 정보를 로드 할 수도 있지만 `FoxMenuManager`가 제공하는 `SaveMenu`, `GetMenuXml`, `WriteMenuXml` 메서드를 통해 로드 된 메뉴 정보를 메뉴 XML로 기록하는 것이 가능합니다.

## 메뉴 확장

`FoxMenuManager`가 관리하는 메뉴 XML, `FoxMenuItem` 객체는 확장이 가능하며 추가적인 속성을 갖도록 커스터마이징이 가능합니다. `FoxMenuItem` 클래스의 `Parse` 메서드와 `WriteXml` 메서드는 virtual 메서드로써 파생 클래스에서 작동 방식을 override 할 수 있습니다. 따라서 `FoxMenuItem` 클래스에서 파생된 클래스를 정의하고 `Parse` 메서드 및 `WriteXml` 메서드를 override 하여 추가적인 속성을 정의할 수 있습니다.

다음은 `FoxMenuItem`에서 파생된 `CustomMenuItem` 클래스의 예를 보여줍니다. 이 클래스는 기본 메뉴 속성 외에 `Data` 속성을 추가하였고 메뉴 XML로부터 데이터를 읽어 들이거나 데이터를 메뉴 XML에 기록하고 있습니다.

```csharp
public class CustomMenuItem : FoxMenuItem
{
    private string _data;
    public string Data
    {
        get { return _data; }
        set { _data = value; }
    }

    public override void Parse(XmlNode xmlNode, bool enableDefault)
    {
        base.Parse(xmlNode, enableDefault);
        // Data 속성 파싱
        GetOptionalStringAttribute(xmlNode, "data", ref _data);
    }

    protected override void WriteXml(XmlElement element)
    {
        base.WriteXml(element);
        // Data 속성 기록 (optional로서 null 인경우 기록하지 않음)
        if (_data != null)
        {
            FoxMenuItem.WriteXmlAttribute(element, "data", _data);
        }
    }

    // ... 기타 코드 생략 ...
}
```

메뉴 확장 기능은 고려해야 할 사항이 많기 때문에 NeoDEEX 기술 지원의 도움을 받는 것이 좋습니다.

---