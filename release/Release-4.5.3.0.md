# NeoDEEX 4.5.3 Change Log

NeoDEEX 4.5.2.x 버전에 대한 변경 사항들 중에서 주목해야 할 사항들을 나열한 것입니다. **이 문서에서 언급되지 않은 변경사항**이 있을 수도 있으며, 이 문서에서 언급된 변경 사항 역시 예고 없이 변경될 수 있습니다.

---

## 4.5.3.0 버전

### Breaking Change

* FoxServiceDialogInfo 클래스의 GetSubMethodType 메서드가 구현 내부 사용용으로 전환되어 public에서 internal으로 변경 되었습니다.

### Fox Query에서 DB의 optional 매개변수 지원

Fox Query에서 데이터베이스의 optional 매개변수를 지원합니다.

* Fox Query의 &lt;parameter&gt; 요소에 optional 속성이 추가되었습니다. 디폴트 값은 false 입니다.

  * optional 속성이 true로 지정되면 Fox Query가 수행될 때 이 매개변수가 제공되지 않더라도 오류가 발생하지 않으며 이 매개변수는 데이터 베이스에 제공되지 않습니다. 따라서 데이터 베이스의 optional 매개변수가 지정하는 기본값이 사용됩니다.

  * Fox Query Editor의 매개변수 편집기에도 optional 컬럼이 추가되었습니다.

```sql
-- SQL SERVER 예제
CREATE PROCEDURE [dbo].[UTP_MergeName]
(
 @ProductID int = 10000,
 @ProductName nvarchar(50) = 'Name'
)
AS
BEGIN
    SELECT CONCAT(@ProductID, @ProductName) AS NAME
END
GO
```

```xml
<!-- Fox query 예제 -->
<procedure id="UTP_MergeName">
    <text>
    UTP_MergeName
    </text>
    <parameters>
    <parameter name="ProductID" dbType="Int" direction="Input" optional="true"/>
    <parameter name="ProductName" dbType="VarChar" direction="Input" optional="true"/>
    </parameters>
</procedure>
```

```cs
// C# 코드 예제
string procedureId = "Product.UTP_MergeName";
FoxSqlDbAccess db = new FoxSqlDbAccess(ConnectionString, "SqlMap");
Dictionary<string, object> parameter = new Dictionary<string, object>();
// ProductID 매개변수 생략 (디폴트 값 사용)
parameter.Add("ProductName", "가나다");
var result = (string)db.ExecuteQueryScalar(procedureId, parameter);
Assert.AreEqual("10000가나다", result);
```

### Fox Biz Service 매개변수 바인딩 기능 개선

비즈 메서드의 매개변수가 IDictionary<string, object>인 경우, FoxBizRequest 객체의 Parameters 컬렉션이 이 매개변수로 직접 바인딩 됩니다.

* 비즈 메서드의 매개변수가 FoxServiceParameterCollection 타입 혹은 IDictionary<string, object> 타입의 매개변수 1개를 사용할 때 FoxBizRequest의 Parameters 속성의 컬렉션이 그대로 전달됩니다.

  * 비즈 메서드는 FoxServiceParameterCollection 혹은 IDictionary<string, object> 타입 매개변수 1개만을 사용해야 합니다. 그리고 매개변수는 ref 혹은 out 타입일 수 없습니다.

  * 비즈 메서드의 리턴타입은 임의의 타입이 사용될 수 있습니다.

```cs
// 비즈 메서드 예제
[FoxBizMethod]
public DataSet MyMethod(IDictionary<string, object> parameters)
{
    var dbAccess = FoxDatabaseFactory.CreateDatabase();
    return dbAccess.ExecuteQueryDataSet("MyFoxml.MyQuery", parameters);
}

// 클라이언트 예제
var request = new FoxBizRequest("MyClass", "MyMethod");
request["ProductId"] = 1;
request["CategoryId"] = 3;
using (var client = FoxClientFactory.CreateBizClient("bizservie.svc"))
{
    var response = client.Execute(request);
    dataGrid.DataSource = response.DataSet.Tables[0];
}
```

### Fox Biz Service 비즈 메서드 커스텀 바인딩 기능 추가

Fox Biz Service가 FoxBizMethodAttribute 가 명시된 메서드를 호출할 때 FoxBizRequest의 Parameters 속성에 기록된 매개변수들을 메서드의 매개변수로 바인딩하는 기능이 개선되었습니다.

* 비즈 메서드의 매개변수와 FoxBizRequest.Parameters 컬렉션을 바인딩하는 역할을 수행하는 매개변수 바인더를 개발자가 직접 개발하여 사용할 수 있습니다.

  * 커스텀 매개변수 바인더를 사용하기 위해서는 FoxBizMethodAttribute 클래스의 Binder 속성을 사용하여 바인더 타입을 명시해 주어야 합니다. 바인더 타입이 명시되지 않으면 Fox Biz Service가 기본으로 제공하는 바인더를 사용하게 됩니다.

  * 바인더는 FoxBizMethodBinder 추상 클래스에서 파생된 클래스이어야 하며 Invoke 메서드를 오버라이드하여야 합니다. 또한, 바인더 클래스는 디폴트 생성자(매개변수가 없는 생성자)를 명시적 혹은 암시적으로 제공해야 합니다.

    * Invoke 메서드는 클라이언트가 비즈 서비스를 호출하여 실제 비즈 메서드가 호출해야 할 때 호출되며 Invoke 메서드 내에서 적절한 매개변수를 사용하여 비즈 메서드를 호출하고 그 결과를 반환해야 합니다.

  * 커스텀 바인더가 명시되었지만 적절한 타입이 아니거나 필요한 생성자를 제공하지 않는 경우, 해당 비즈 메서드는 유효하지 않은 것으로 판단되어 비즈 서비스를 통해 호출되지 않고 예외를 유발합니다. 유효 하지 않은 커스텀 바인더는 비즈 서비스 구동(FoxBizServiceConfig.Configure 호출 시점)시 로그에 기록됩니다.

```cs
// 레거시 코드가 커스텀 매개변수 타입을 사용하고 이를 전환하기 어려운 경우,
// 커스텀 메서드 바인더를 통해 매개변수 전환이 가능합니다.
// 다음은 이러한 예를 보여주고 있습니다.
class TestCustomBinder : FoxBizMethodBinder
{
    public override FoxBizResponse Invoke(MethodInfo methodInfo, object targetInstance, FoxBizRequest request)
    {
        var parameterSet = new ParameterSet();
        foreach(var p in request.Parameters)
        {
            parameterSet.Add(p.Key, p.Value);
        }
        var args = new object[] { parameterSet };
        var result = methodInfo.Invoke(targetInstance, args);
        var response = result as FoxBizResponse;
        if (response == null)
        {
            response = new FoxBizResponse(true, result);
        }
        FoxBizMethodBinder.GetOutputParameters(pis, args, response.Parameters);
        return response;
    }
}

[FoxBizClass]
public class TestBizClass : TestBizClassBase
{
    // 레거시 코드를 Fox Biz Service를 통해 호출하기 위해 커스텀 메서드 바인더 사용
    [FoxBizMethod(typeof(TestCustomBinder))]
    public int CustomBinderTest(ParameterSet parameters)
    {
        // ... 생략 ...
    }
}
```

### Fox Biz Service 초기화 방식 개선

Fox Biz Service는 FoxBizServiceConfig.Configure 메서드를 어플리케이션 시작 시점(대개 global.asax.cs)에서 호출함으로써 초기화 됩니다. 이 초기화 과정에서 다양한 개선이 이루어 졌습니다.

* Configure 호출 시 구성 설정 파일의 &lt;modules&gt; 요소에 명시된 모듈을 읽어 들일 때 발생할 수 있는 오류에 대해 예외 발생 여부를 제어할 수 있습니다.

  * &lt;modules&gt; 요소에 throwOnError 속성이 추가되었습니다. 이 속성의 디폴트 값은 true로서 모듈들을 읽어 초기화 할 때 발생하는 오류(FileNotFound 등)들에 대해 예외를 발생시킵니다.

  * throwExeption 속성이 false 이면 초기화 시 발생하는 예외는 무시되며 오류를 발생한 모듈은 로드 되지 않습니다.

```xml
  <service>
    <bizService>
      <modules throwOnError="false">
        <module name="TheOne.ServiceModel.Test" />
        <module name="SomeModule" />
        <module name="OtherModule" />
      </modules>
    </bizService>
  </service>
```

* 비즈 서비스 초기화 시 기록되는 로그에 보다 상세한 정보가 기록됩니다.

  * &lt;modules&gt; 요소의 throwOnError 속성 값에 무관하게 발생한 예외에 대해서는 비즈 서비스 로그에 기록됩니다.

  * 모듈에서 발견된 비즈 클래스들에 대한 추가적인 정보(클래스 이름, 비즈 메서드 개수, 클래스 ID)가 표시됩니다.

  * 중복 ID를 가진 비즈 클래스, 비즈 메서드에 대한 정보가 로그에 기록됩니다.

  * 비즈 모듈 로드 과정에서 오류가 발견되면 오류가 로그에 기록됩니다. 예외 발생 여부는 throwOnError 속성 설정 값에 의해 결정됩니다.

```txt
FoxBizService: Start configuring ......
starting to load predefined biz modules:
Loading biz module: TheOne.ServiceModel.Test
    loaded assembly successfully: TheOne.ServiceModel.Test, Version=4.5.0.0, Culture=neutral, PublicKeyToken=6895727a3cc10e00
Loaded biz class: TheOne.ServiceModel.Test.Biz.TestBizClassBase, 1 biz methods, id=TheOne.ServiceModel.Test.Biz.TestBizClassBase
Loaded biz class: TheOne.ServiceModel.Test.Biz.TestBizClass, 13 biz methods, id=TheOne.ServiceModel.Test.Biz.TestBizClass
Loaded biz class: TheOne.ServiceModel.Test.Biz.TestDerivedBizClass, 2 biz methods, id=CustomBizClass
Loaded biz class: TheOne.ServiceModel.Test.Biz.TestFarDerivedBizClass, 3 biz methods, id=CustomDerivedBizClass
Loaded biz class: TheOne.ServiceModel.Test.Biz.TestBizComponent, 3 biz methods, id=TheOne.ServiceModel.Test.Biz.TestBizComponent
Loaded biz class: TheOne.ServiceModel.Test.Biz.BizClass, 8 biz methods, id=비즈클래스
Loading biz module: InvalidModule
FoxBizService: cannot load assembly, InvalidModule : System.IO.FileNotFoundException: 파일이나 어셈블리 'InvalidModule' 또는 여기에 종속되어 있는 파일이나 어셈블리 중 하나를 로드할 수 없습니다. 지정된 파일을 찾을 수 없습니다.
파일 이름: 'InvalidModule'
   위치: System.Reflection.RuntimeAssembly._nLoad(AssemblyName fileName, String codeBase, Evidence assemblySecurity, RuntimeAssembly locationHint, StackCrawlMark& stackMark, IntPtr pPrivHostBinder, Boolean throwOnFileNotFound, Boolean forIntrospection, Boolean suppressSecurityChecks)
   위치: System.Reflection.RuntimeAssembly.nLoad(AssemblyName fileName, String codeBase, Evidence assemblySecurity, RuntimeAssembly locationHint, StackCrawlMark& stackMark, IntPtr pPrivHostBinder, Boolean throwOnFileNotFound, Boolean forIntrospection, Boolean suppressSecurityChecks)

... 생략 ...

FoxBizService: End configuring ......
```

### 메시지 박스 Owner 자동 탐지 기능 추가

FoxMessageBox를 통해 메시지 박스를 표시할 때 Owner가 명시적으로 지정되지 않거나 null로 지정된 경우 Owner는 데스크톱 화면이 됩니다. 이때 Windows 시스템은 종종 메시지 박스가 최상단이 아니게 되어 다른 윈도우 뒤로 숨는 현상이 발생합니다. 이 문제를 해결하기 위해 Owner가 지정되지 않은 경우, 경험적 방법을 통해 Owner를 자동으로 찾는 기능이 추가되었습니다.

* FoxMessageBox 클래스에 ForceResolveOwner 속성이 추가되었습니다. 이 속성이 true로 지정되면 FoxMessageBox 클래스는 Owner가 지정되지 않거나 null 로 지정된 경우, 경험적 방법을 통해 Owner를 찾으려고 시도합니다. 이 속성의 기본값은 true 입니다.

* ForceResolveOwner 속성이 true 인 경우 FoxMessageBox 클래스가 자동으로 지정하는 Owner 윈도우는 다음과 같습니다(순서대로).

  * 예외 메시지 박스의 Owner로 사용되도록 FoxExceptionHandler.MainControl 속성에 지정된 윈도우(null이 아닌 경우에만)

  * Form.ActiveForm 속성에 지정된 윈도우(null이 아닌 경우에만)

  * Application.OpenForms[0] 값이 나타내는 윈도우(Application.Run 호출에 전달된 폼)

  * 위의 모든 경우가 null 이라면 데스크톱 윈도우 사용(기본 행동)

* ForceResolveOwner 속성이 true 인 상황에서 명시적으로 메시지 박스의 Owner를 데스크톱 윈도우로 지정하기 위해서 FoxMessageBox.DesktopWindow를 Owner 매개변수로 사용할 수 있습니다.

* **주의) 비록 ForceResolveOwner 속성을 통해 Owner를 자동으로 설정할 수 있지만 이 기능에 의존하는 것보다는 메시지 박스를 사용할 때 명시적으로 Owner를 지정하는 것이 권장되는 코딩 패턴입니다**.

```cs
// 예외 메시지 박스를 위한 메인 폼 지정
FoxExceptionHandler.MainControl = frmMain;
// 메시지 박스 owner 자동 설정 기능 활성화
FoxMessageBox.ForceResolveOwner = true;
// 자동으로 owner가 frmMain으로 지정됩니다.
FoxMessageBox.Show("Message box...");
// 명시적으로 owner를 null(데스크톱)으로 지정합니다.
FoxMessageBox.Show(FoxMessageBox.Desktop, "Message box...");
```

## 기타 버그 수정

* BUG 679: FoxAccessControlManager가 제공하는 RequiredAccess 속성의 UI 에디터에서 cast 오류가 발생함.

  * 컨트롤 권한을 디자인 타임(폼 디자이너)에서 지정할 때 에디터를 구동하면 type cast 오류가 발생했습니다. 이 문제가 해결되었습니다.

* BUG 76: 데이터베이스 브라우저에서 Drag/Drop으로 Fox query 내용을 수정하여도 Foxml 파일이 변경되지 않음.

  * Fox Query Editor의 stand alone 버전에서 데이터베이스 브라우저에서 쿼리 혹은 저장 프로시저를 drag & drop을 사용하여 새로운 쿼리를 만들더라도 파일이 변경되지 않은 것으로 간주되어 명시적으로 저장을 하지 않으면 경고가 발생하지 않았습니다. 이 문제점이 해결되었습니다.

---
