# NeoDEEX 4.5.6 Change Log

NeoDEEX 4.5.6.x 버전에 대한 변경 사항들 중에서 주목해야 할 사항들을 나열한 것입니다. **이 문서에서 언급되지 않은 변경사항**이 있을 수도 있으며, 이 문서에서 언급된 변경 사항 역시 예고 없이 변경될 수 있습니다.

> 주의: 4.5.6.0 버전은 릴리스 되지 않았으며 4.5.6.1 버전이 릴리스 되었습니다.

---

## Breaking Change

이번 릴리스에는 하위 호환성을 해치는 변경 사항이 없습니다.

## Fox Biz/Data Service 기능 개선

* Fox Biz Service의 비즈 모듈 로드 기능이 개선되었습니다.

  * Fox Biz Service의 비즈 모듈 목록에 어셈블리 경로와 와일드 카드를 포함하는 파일명을 명시할 수 있습니다.

    * `<module>` 요소에 `name` 속성에 **어셈블리 파일명**을 명시하고 `directory` 속성에 어셈블리 디렉터리를 명시하면 해당 어셈블리 파일(DLL)을 비즈 모듈로서 로드 합니다.

      ```xml
      <modules>
        <module name="MyBizModule1.dll" directory='bin' />
      </modules>
      ```

    * `directory` 속성이 명시되지 않으면 기존과 동일하게 name 속성은 파일 이름이 아닌 어셈블리 이름으로 인식되어 집니다. 어셈블리 이름이므로 .dll 과 같은 확장자를 붙이면 오류가 발생합니다.

    * `directory` 속성이 명시된 경우, `name`속성에 와일드 카드(* 혹은 ? 포함)를 사용할 수 있습니다.

    * `directory` 속성은 어플리케이션 도메인으로부터의 상대 경로 혹은 절대 경로로 설정이 가능합니다.

  * 와일드 카드를 포함하는 `name` 속성과 `directory` 속성을 사용하면 여러 비즈 서비스 웹 어플리케이션이 하나의 NeoDEEX 구성 파일을 공유하는 것이 가능합니다. 여러 비스 서비스 웹 어플리케이션이 동일한 구성을 갖지만 비즈 모듈 목록이 다른 경우 응용이 가능합니다.

    ```xml
    <modules>
      <!-- CA 업무 비즈 모듈들 -->
      <module name="CA.BIZ.*.dll" directory='MODULES\CA' />
      <!-- MM 업무 비즈 모듈들 -->
      <module name="MM.BIZ.*.dll" directory='MODULES\MM' />
    </modules>
    ```

  * 중복된 ID를 가진 비즈 메서드들을 포함하는 비즈 클래스가 중복된 비즈 메서드 중 최초 하나만을 포함하여 등록되도록 변경 되었습니다.

    > 이전 버전에서는 중복된 ID를 가진 비즈 메서드가 존재하면 비즈 클래스는 등록되지 않았었습니다.

* Fox Biz/Data Service 관련 코드들이 사용자 정의 로그를 기록하거나 Request 객체에 접근할 수 있는 `FoxBizServiceContext/FoxDataServiceContext` 클래스가 추가 되었습니다.

  * Fox Biz Service의 비즈 메서드에서는 `FoxBizServiceContext.Current` 속성을 사용하여 현재 수행중인 FoxBizService 문맥에 접근이 가능합니다.

    * `FoxBizServiceContext.WriteLog` 메서드를 통해 NeoDEEX 구성 파일에 지정된 비즈 서비스 로그에 로그를 기록할 수 있습니다.

    * `FoxBizServiceContext.Request` 속성을 통해 클라이언트에서 요청된 `FoxBizRequest` 객체에 접근이 가능합니다.

      ```cs
      [FoxBizMethod]
      public void MyBizMethod(string s)
      {
        var ctx = FoxBizServiceConfig.Current;
        ctx.WriteLog(FoxLogLevel.Verbose, $"Client IP : {ctx.Request.ClientInfo.IP}");
      }
      ```

  * Fox Data Service와 관련된 코드에서 `FoxDataServiceContext.Current` 속성을 사용하여 현재 수행중인 `FoxDataService` 문맥에 접근이 가능합니다.

    * `FoxDataServiceContext.WriteLog` 메서드를 통해 NeoDEEX 구성 파일에 지정된 비즈 서비스 로그에 로그를 기록할 수 있습니다.

    * `FoxDataServiceContext.Request` 속성을 통해 클라이언트에서 요청된 `FoxDataRequest` 객체에 접근이 가능합니다.

* Fox Biz Service의 초기화시 기록되는 로그 메시지가 좀더 명확한 상황을 표시하도록 개선 되었습니다.

  * 비즈 모듈, 비즈 모듈 내의 비즈 클래스 초기화 메시지에 대해 인덴트가 사용되어 모듈 로드 오류 시 어떤 모듈, 어떤 클래스에서 오류가 발생했는지 알기 편하도록 변경 되었습니다.

    ```log
    I Start configuring ......
    I Starting to load predefined biz modules:
    I   Loading biz module: TheOne.ServiceModel.Test
    I     Loading assembly successfully: TheOne.ServiceModel.Test, ...
    I     Loading biz class: TheOne.ServiceModel.Test.Biz.TestBizClassBase, 1 biz methods, ...
    I     Loading biz class: TheOne.ServiceModel.Test.Biz.TestBizClass, 18 biz methods, ...
          ......
    I   Loading biz module: TheOne.ServiceModel.Test3
    E     > ERROR! cannot load assembly, TheOne.ServiceModel.Test3
    ```

* Fox Biz Service의 매개변수에 POCO 타입이 사용되는 경우 JSON.NET의 기본 직렬화기 (혹은 등록된 JsonConverter를 통해) POCO 타입이 직렬화될 수 있습니다.

* Decimal 타입의 유효 숫자가 잘리지 않도록 decimal 타입을 문자열로 직렬화할 수 있는 구성 설정이 추가되었습니다.

  * Fox Biz/Data Service가 사용하는 JSON.NET은 json 문자열에 실수가 사용되면 기본적으로 `double` 타입으로 역직렬화를 수행합니다. `decimal` 타입이 json으로 직렬화되어 전송되고 수신 측에서 역직렬화가 `double` 타입으로 역직렬화 되는 경우 유효숫자 개수가 적은 `double` 타입에서 유효숫자가 잘릴 수 있습니다.

    ```json
    "parameters" : {
      "param1" : {
        "type" : decimal,
        "value" : 100000000000006001   <= 유효 숫자가 잘리는 문제 발생
      }
    }
    ```
  
  * 이러한 문제(BUG-746)를 해결하기 위해 `double` 타입이 직렬화 될 때 문자열로서 직렬화 되도록 구성 설정의 `<json>` 요소에 `writeDecimalAsString` 속성이 추가되었습니다.

    ```xml
    <json writeDeciamlAsString="true" />
    ```

    * `writeDecimalAsString` 속성이 `true` 로 지정되면 `decimal` 타입이 직렬화 될 때 문자열로서 직렬화 됩니다.

    ```json
    "parameters" : {
      "param1" : {
        "type" : decimal,
        "value" : "100000000000006001"
      }
    }
    ```

    * `writeDecimalAsString` 속성의 기본값은 `false` 입니다.

  * 문자열로 직렬화된 `decimal` 타입의 데이터는 유효숫자가 잘리지 않고 정상적으로 역직렬화 됩니다.

* `DataSet`의 이름이 JSON으로 직렬화 될 수 있도록 구성 설정이 추가되었습니다.

  * Fox Biz/Data Service에서 `DataSet`이 매개변수/결과값으로 사용되는 경우, 기본적으로 `DataSet`의 이름은 직렬화 되지 않습니다. 하지만 일부 어플리케이션에서 `DataSet`의 이름을 사용하는 경우가 있기 때문에 NeoDEEX 구성 설정을 통해 `DataSetName` 속성을 직렬화할 수 있습니다.

    * `<appSettings>` 요소에 `FoxJsonUseDataSetName` 키/값을 true로 지정하면 `DataSet` 객체의 `DataSetName` 속성 값이 직렬화/역직렬화 됩니다.

    ```xml
    <appSettings>
      <add name="FoxJsonUseDataSetName" value="true">
    </appSettings>
    ```

  * `FoxJsonUseDataSetName` 키/값은 클라이언트/서비스에 모두 동일하게 설정해야 합니다.

* `FoxBizClassAttribute` 클래스에 비즈 메서드의 디폴트 바이더를 지정할 수 있는 `DefaultBinderType` 속성이 추가되었습니다.

  * Fox Biz Service 호출시 `FoxBizRequest` 객체의 매개변수를 비즈 메서드의 매개변수로 매핑하고 호출하는 역할을 바인더(binder)가 수행합니다. 바인더는 `FoxBizMethodAttribute` 클래스의 `BinderType` 속성을 통해 지정이 가능하며, 지정되지 않은 경우 디폴트 바인더(`FoxFastBinder` 혹은 `FoxDefaultBinder`)가 사용됩니다.

  * 비즈 클래스의 모든 메서드에서 적용될 수 있는 디폴트 바인더를 `FoxBizClassAttribute` 클래스의 `DefaultBinderType` 속성을 통해 지정이 가능합니다. `DefaultBinderType` 속성이 지정되지 않거나 `null`로 설정된 경우, 디폴트 바인더가 사용됩니다.

  * `FoxBizMethodAttribute` 클래스의 `BinderType` 속성이 명시된 경우 `DefaultBinderType` 속성의 설정값은 무시되고 `BinderType` 속성값이 사용됩니다.

* FoxRestBizClient/FoxRestDataClient 클래스의 동기 메서드들 호출이 예외를 발생하는 경우, 호출 스택이 예외가 발생한 위치를 정확하게 표시하도록 개선되었습니다.

## Fox WinForm 개선 사항

Fox WinForm에서 다음 몇가지 사항이 개선되었습니다.

* 프로그래스 표시까지 지연 시간을 설정할 수 있도록 `FoxProgressDialogBase` 클래스에 `ShowDelay` 속성이 추가 되었습니다.

  * `FoxAsyncProxy` 클래스를 사용하여 비동기 작업(서버 호출)이 수행될 때 자동으로 나타나는 프로그래스 UI가 깜박이는 현상이 발생합니다. 이는 비동기 작업이 짧은 시간(약 100msec 전후) 동안 수행되는 경우 입니다. 이를 해결하기 위해 `ShowDelay` 속성 값을 지정하면 프로그래스 UI는 `ShowDelay` 속성이 지정한 시간(msec)이 지난 후에만 나타나게 됩니다.

  * `ShowDelay` 속성의 기본값은 0 이며 지연 없이 즉시 프로그래스 UI가 표시됩니다.

* `FoxProgressDialog` 클래스에 의해 표시되는 프로그래스 UI에 소요 시간 표시 여부를 결정하는 `ShowElapsed` 속성이 추가되었습니다.

  * `FoxProgressDialog` 클래스는 프로그래스 UI를 표시하며 3초 이상 소요되는 경우, 현재 소요되고 있는 시간을 0.1초 단위로 프로그래스 UI 하단에 표시합니다. `ShowElapsed` 속성을 통해 이 소요 시간을 표시할 지 여부를 결정할 수 있습니다.

  * `ShowElpased` 속성의 기본값은 true 입니다.

```cs
// FoxForm에서 파생된 클래스에서 GetProgressDialog 메서드 오버라이드
protected override IFoxProgressDialog GetProgressDialog()
{
    var dlg = base.GetProgressDialog() as FoxProgressDialog;
    if (dlg != null)
    {
        // 300msec 이상 수행되는 작업에 대해서만 프로그래스 UI 표시
        dlg.ShowDelay = 300;
        // 소요 시간 표시하지 않음
        dlg.ShowElased = false;
    }
    return dlg;
}
```

* `FoxProgressDialog` 클래스에 의해 표시되는 프로그래스 UI가 마우스 캡처를 해제하는 기능을 제어하는 `ReleaseMouseCapture` 속성이 추가되었습니다.

  * `FoxProgressDialog`의 의해 표시되는 프로그래스 UI는 마우스 캡처가 수행된 경우 이를 해제하고 복원합니다. 마우스 캡처를 사용하는 3rd Party 일부 컨트롤이 프로그래스 UI와 충돌을 일으켜 정상 작동하지 않는 경우, `ReleaseMouseCapture` 속성을 false로 지정하여 프로그래스 UI가 마우스 캡처를 해제하지 않도록 설정할 수 있습니다.

  * `ReleaseMouseCapture` 속성의 기본값은 true 입니다. `FoxProgressDialogBase` 클래스의 `ReleaseMouseCaptureDefault` 정적(static) 속성을 통해 `RelaseMouseCapture` 속성의 기본값을 제어할 수 있습니다.

## Fox Transaction 기능 개선

* `FoxTransactionController` 특성에 지정할 수 있는 컨트롤러 타입으로 `FoxTransactionControllerKind` 열거 타입에 `RootContext` 옵션이 추가되었습니다.

  * `FoxTransactionController` 특성은 트랜잭션 제어에 사용할 컨트롤러의 종류(`FastTransaction`, `TranactionScope`, `LocalTransaction` 등)를 `FoxTransactionControllerKind` 열거 타입을 통해 지정할 수 있습니다. 트랜잭션 클래스들은 동일한 트랜잭션 컨트롤러를 사용해야만 합니다. 하지만 여러 클래스에 의해 호출되는 클래스는 특정한 한 종류의 트랜잭션 컨트롤러를 지정할 수 없는 경우도 있습니다. 예를 들어 Fast 트랜잭션 컨트롤러를 사용하는 클래스와 Local 트랜잭션 컨트롤러를 사용하는 클래스가 모두 호출하는 클래스의 트랜잭션 컨트롤러를 설정하기 어려웠습니다. 이 때 `RootContext` 옵션을 사용하면 호출자의 트랜잭션 컨트롤러를 사용하도록 지정할 수 있습니다.

  * 호출자가 트랜잭션 컨트롤러를 가지고 있지 않은 경우, 디폴트 컨트롤러인 Fast 트랜잭션 컨트롤러가 사용됩니다.

  * 대부분의 경우, BIZ 컴포넌트는 명시적으로 Fast 혹은 Local 트랜잭션 컨트롤러를 지정하고 DAC 컴포넌트는 `RootContext` 옵션을 지정하면 됩니다.

    ```cs
    [FoxTransactionController(FoxTransactionControllerKind.RootContext)]
    public class MyDacBase : FoxDacBase
    {
      ......
    }
    ```

* COM+ 호환성을 위한 `FoxTransactionComplusCompatible` 구성 설정이 추가되었습니다.

  * NeoDEEX 구성 설정의 `<appSettings>` 요소에 `FoxTransactionComplusCompatible` 키 값을 true 로 지정되면 COM+ 호환성이 활성화 됩니다.

  * COM+ 호환성이 활성화되면 자동 트랜잭션이 사용된 경우(`FoxAutoComplete=true`) SetComplete/SetAbort 메서드 호출이 예외를 발생시키지 않습니다.

    * Fox Transaction은 자동 트랜잭션이 사용된 경우(`FoxAutoComplete` = true), 트랜잭션의 커밋/롤백 여부는 오직 예외 발생 여부에 의해 결정됩니다(COM+ 동일). 따라서 자동 트랜잭션이 사용되는 경우 `SetComplete`/`SetAbort` 메서드가 호출되면 `InvalidOperationException` 예외를 발생하여 잘못된 코딩 패턴을 방지합니다.

    * COM+는 자동 트랜잭션이 사용된 경우, SetComplete/SetAbort 메서드 호출은 아무런 효과가 없으며 예외도 발생하지 않습니다.

    * COM+ 기반의 어플리케이션을 Fox Transaction으로 마이그레이션 하는 경우 SetComplete/SetAbort 호출 시 발생하는 예외가 마이그레이션을 어렵게 할 수 있습니다. 이 경우 COM+ 호환성을 활성화함으로써 마이그레이션을 보다 쉽게 할 수 있습니다. 하지만 **Fox Transaction의 권장 코딩 패턴 상 COM+ 호환성 설정은 권장하지 않습니다.**

  * 수동 트랜잭션 사용 시(`FoxAutoComplete=false`) COM+의 트랜잭션 기본 결과값은 커밋이며 Fox Transaction은 롤백 입니다. COM+ 호환성이 활성화되면 Fox Transaction의 트랜잭션 기본 결과값이 커밋으로 변경됩니다.

## Fox Updater 기능 개선

* Fox Updater가 구동하는 런처에 커맨드 라인 매개변수를 설정할 수 있도록 `FoxUpdateSettings` 클래스에 `LauncherArguments` 속성이 추가되었습니다.

* Fox Updater가 런처를 관리자 권한으로 구동할 수 있도록 `FoxUpdateClient` 클래스에 `IsAdministrator` 메서드와 `RestartWithAdminRole` 메서드가 추가되었습니다.

  * `IsAdministrator` 메서드는 현재 Updater 클라이언트가 관리자 권한인가 확인하는 메서드 입니다. `RestartWithAdminRole` 메서드는 Updater 클라이언트를 관리자 권한으로 다시 구동하는 메서드 입니다.

  * Updater 클라이언트 코드는 `IsAdministrator` 메서드를 사용하여 현재 업데이터가 관리자 권한인가 확인하고 관리자 권한이 아니라면 `RestartWithAdminRole` 메서드를 호출하여 관리자 권한으로 Updater 클라이언트를 다시 구동하고 업데이트 및 런처 구동을 수행하면 됩니다.

    ```cs
    public void Main()
    {
        if (FoxUpdateClient.IsAdministrator() == false)
        {
           FoxUpdateClient.RestartWithAdminRole();
           return;
        }
        ... 업데이터 일반 코드 ...
    }
    ```

  > 참고) Updater가 런처를 구동할 때, 환경 변수를 통해 매개변수를 전달하기 때문에 `runas` 동사(verb)를 사용할 수 없습니다. 따라서 Updater를 스스로 관리자 권한으로 재 구동하고 런처를 구동하는 방식을 사용합니다.

## 기타 변경 사항

* `FoxClientFactory` 클래스의 `GetTargetUrl` 메서드가 `public` 메서드로 전환되었습니다. 이 메서드는 주어진 상대 Uri에 주소 맵의 항목을 적용하여 절대 Url을 구성하여 반환하는 메서드 입니다.

## 버그 수정

* BUG 734: `FoxDbParameterCollection.AddWithValue`에 `DBNull.Value` 값을 추가하면 예외가 발생함.

  * `FoxDbParameterCollection` 혹은 그 파생 클래스의 `AddWithValue` 메서드의 값으로 매개변수 이름과 `DBNull.Value`를 추가하면 Enum Parse 예외가 발생하는 버그를 수정하였습니다.

* BUG 735: 스칼라 값을 반환하는 Biz 메서드에서 cast 오류가 발생함.

  * Vaule 타입을 반환하는 일부 비즈 메서드에서 `ArgumentException`이 발생하는 버그를 수정하였습니다.

* BUG 746: JSON 역직열화 시 `decimal` 타입에 대해 유효숫자가 잘림.

  * 유효숫자가 많은 `decimal` 타입의 JSON 문자열에 포함된 경우, 역직렬화 시 일부 유효숫자가 잘리는 버그를 수정하였습니다. 수정을 위해 `writeDecimalAsString` 구성 설정을 사용해야 합니다.

* BUG 749: `FoxDbProfile`의 InlineQuery에서 매개변수 치환이 부적절함.

  * FoxDbProfile에 의해 기록되는 로그의 InlineQuery에서 매개변수의 이름이 `p1`, `p11`, `p111`과 같이 일부 중첩되는 경우 매개변수 값이 부적절하게 치환되는 버그를 수정하였습니다.

* BUG 763: FoxUserInfoContext 직렬화 도중 InvalidOperationException이 발생함.

  * FoxRestBizClient/FoxRestDataClient 호출 등 일부 상황에서 FoxUserInfoContext 직렬화 시 System.InvalidOperationException(열거자를 인스턴스화한 후 컬렉션이 수정되었습니다) 예외가 발생하는 버그를 수정하였습니다.

---
