# NeoDEEX 4.5.4 Change Log

NeoDEEX 4.5.4.x 버전에 대한 변경 사항들 중에서 주목해야 할 사항들을 나열한 것입니다. **이 문서에서 언급되지 않은 변경사항**이 있을 수도 있으며, 이 문서에서 언급된 변경 사항 역시 예고 없이 변경될 수 있습니다.

---

## 주의 사항

**이번 버전의 변경 사항은 Binary 수준에서 호환되지 않을 수 있습니다.** 반드시 사전 테스트를 수행하고 오류가 발생하는 경우, NeoDEEX를 참조하는 모듈들을 다시 컴파일하여야 합니다. 특히, 운영 배포 중인 프로덕션 환경에서는 더욱 주의가 필요합니다.

---

## Breaking Change

* `FoxDataRequest` 객체의 `Parameters` 컬렉션에 매개변수를 추가하고 값으로 `null`을 할당하면 자동으로 `DBNull` 값으로 변경합니다. ([SetNullToDBNull 기능](#SetNullToDBNull%20기능%20추가) 참조)

  * 이 기능이 호환성 문제를 일으킨다면 자동 변환 기능을 off 할 수 있습니다.

* `FoxModuleLoader` 클래스의 `protected` 멤버인 `CreateView` 메서드의 이름이 `CreateViewCore` 메서드로 변경되었습니다.

* 다음 변경 사항으로 인해 `TheOne.ServiceModel.dll`을 사용하는 모듈을 다시 컴파일하고 배포해야 할 수도 있습니다.

  * `FoxDataRequest` 클래스의 `CommandTimeout` 속성 및 `TransactionTimeout` 속성의 타입이  `int` 에서 `int?` 타입으로 변경되었습니다.

  * `FoxDataRequestCollection` 클래스의 `CommandTimeout` 속성의 타입이 `int` 에서 `int?` 타입으로 변경되었습니다.

* **BUG-174의 수정으로 인해 Updater 서버에서 기본 디렉터리(baseDir)의 하위 디렉터리에서만 다운로드가 가능하도록 수정되었습니다. 이로 인해 기존 Updater 서버 설정이 다운로드 오류를 유발할 수 있습니다. 다음 내용을 참고하여 구성 설정을 수정해야 합니다.**

    [Updater 설정 문제 해결](HotFix-4.5.3.3.md#문제-해결)

* `FoxDbProfileInfo` 클래스이 `GUID` 속성이 `InfoId` 속성으로 대체되었습니다.

  * `InfoId` 속성은 기존 `GUID` 속성과 동일한 값을 반환합니다.

  * `GUID` 속성은 `Obsolete` 표시되었으며 다음 버전에서 제거될 것입니다.

  * InfoId 속성 값에 GUID를 보다 명확히 하기 위해 { } 문자가 추가되었습니다.

    * 이전 아이디 값: DB-C0033F84-85DD-4546-8BAC-85C472B9594E

    * 현재 아이디 값: DB-{C0033F84-85DD-4546-8BAC-85C472B9594E}

* Fox Db Profile에 의해 측정되는 성능 문맥(`FoxPerformanceContext`)의 이름이 `FoxDbProfileInfo` 객체의 `InfoId`를 반영하도록 수정되었습니다.

## SetNullToDBNull 기능 추가

`FoxServiceParameterCollection` 클래스에 `null` 값ㅇ르 `DBNull` 값으로 변경하는 기능이 추가되었습니다.

* `FoxServiceParameter` 클래스에 `SetNullToDBNull` 메서드가 추가되었습니다. 이 메서드를 호출하면 `FoxServiceParameter.Value` 값이 `null` 인 경우, `DBNull`로 변경이 됩니다.

* `FoxServiceParameterCollection` 클래스에 `SetNullToDBNull` 메서드가 추가되었습니다. 이 메서드가 호출되면 컬렉션에 포함된 매개변수들 중 값들을 검사하여 `null` 값을 모두 `DBNull`로 변경합니다.

* `FoxServiceParameterCollection` 클래스이 `UseDBNull` 속성이 추가되었습니다(디폴트 false). 이 속성이 `true`로 설정되면 컬렉션에 포함된 매개변수에 `null` 값이 할당되면 자동으로 `null`을 `DBNull`로 변경합니다.

  > `FoxDataRequest` 클래스가 생성하는 `FoxServiceParameterCollection` 객체는 기본적으로 `UseDBNull` 속성이 `true`로 설정됩니다. 이 기능이 호환성 문제를 일으킨다면 NeoDEEX 구성 설정에 다음과 같은 설정을 사용하여 `SetNullToDBNull` 기능을 해제할 수 있습니다.

    ```xml
    <appSettings>
      <add name="DataRequestUseDBNull" value="true"/>
    </appSettings>
    ```  

## Fox Query: Ambient Value Parameter

Fox Query에 환경 값 매개변수(Ambient Value Parameter)을 설정할 수 있는 기능이 추가 되었습니다. 환경 값 매개변수는 쿼리를 수행할 때 명시적으로 매개변수 값이 제공되지 않더라도 환경 값 소스로부터 값을 읽어 매개변수로 사용하는 기능 입니다. 상세한 내용은 [환경 값 매개변수 항목](https://github.com/NeoDEEX/manual/blob/master/data/foxquery/ambient.md)을 참고 하십시오.

## Fox Biz/Data Service 성능 로그 지원

Fox Biz/Data Service가 성능 로그를 지원합니다. Fox Biz/Data Service는 이전부터 성능 활동(`FoxPerformanceActivity`)과 성능 문맥(`FoxPerformanceContext`)을 통해 성능 측정을 수행했습니다. 이렇게 측정된 성능 정보는 진단 기능을 통해 클라이언트에게 반환될 수 있었습니다. 4.5.4 버전부터 측정된 성능 정보를 로깅을 통해 기록할 수 있게 되었습니다.

* Fox Biz/Data Service 성능 로그를 위한 구성 설정이 추가되었습니다.

  * `<bizService>` 요소 및 `<dataService>` 요소에 `<perfLog>` 요소가 추가되었습니다.

    * `enable` 속성은 성능 로그 기록 여부를 결정합니다.

    * `loggerName` 속성은 성능 로그를 기록할 로거의 이름입니다.

  * `FoxBizServiceConfig` 및 `FoxDataServiceConfig` 클래스에 `EnablePerfLog` 정적 속성 및 `PerfLoggerName` 정적 속성이 추가되었습니다. 이 속성은 구성 설정의 설정 값을 반영하며 이 속성들을 통해 코드에 의한 로깅 설정도 가능합니다.

* `FoxBizService` 및 `FoxDataService` 클래스에 `EnablePerfLog` 인스턴스 속성 및 `PerfLoggerName` 인스턴스 속성이 추가되었습니다. 이 속성들의 기본값은 `FoxBizServiceConfig` 및 `FoxDataServiceConfig` 클래스의 `EnablePerfLog` 및 `PerfLoggerName` 속성 값으로 초기화 되며 개별 서비스 단위로 이 설정을 변경할 수 있습니다.

* `FoxBizRequest` 혹은 `FoxDataRequest` 클래스의 `Diagnostics` 속성에 `SuppressPerfLogWirte` 플래그를 추가하여 해당 호출에 대해서만 성능 로그를 기록하지 않도록 구성할 수도 있습니다.

다음은 NeoDEEXX 구성 설정을 통해 Fox Data Service의 성능 정보를 표시하는 예를 보여줍니다.

```xml
<service>
  <dataService>
    <perfLog enable='true' loggerName='PerformanceLog' />
  </dataService>
</service>
<logging>
  <loggers>
    <logger name="PerformanceLog" providerType="TheOne.Diagnostics.Loggers.FoxTextFileLoggerProvider">
      <property name="directory" value="D:\Logging"/>
      <property name="fileprefix" value="PerformanceLog"/>
    </logger>
  </loggers>
</logging>
```

> `<bizService>` 요소에 `<perfLog>` 요소를 추가하여 Fox Biz Service에도 동등한 성능 로그를 남길 수 있습니다.

위 설정은 다음과 같은 성능 로그를 표시합니다.

```log
I 2019-09-09 03:21:44.48740 [PerformanceLog] Performance Activity [HOMEWORK:15264:FoxDataService;DataAccessLayer] elapsed=0.26 time=03:21:44:4854 id=e74f5700-5aba-4acd-86c5-aa6b694efab5
    ExecuteDataSet inclusive=0.26 exclusive=0.06 time=03:21:44:4854 id=45031 parent=-1
        FoxQuery inclusive=0.19 exclusive=0.07 time=03:21:44:4854 id=45032 parent=45031
            FoxDbAccess inclusive=0.12 exclusive=0.12 time=03:21:44:4854 id=45033 parent=45032
I 2019-09-09 03:21:44.48840 [PerformanceLog] Performance Activity [HOMEWORK:15264:FoxDataService;DataAccessLayer] elapsed=0.35 time=03:21:44:4874 id=fbcaad17-a73e-43a1-960d-073bf5084892
    ExecuteDataSet inclusive=0.35 exclusive=0.08 time=03:21:44:4874 id=45034 parent=-1
        FoxQuery inclusive=0.27 exclusive=0.08 time=03:21:44:4874 id=45035 parent=45034
            FoxDbAccess inclusive=0.19 exclusive=0.19 time=03:21:44:4874 id=45036 parent=45035
```

> 성능 로그를 DB에 기록하기 위해서는 `FoxPerformanceAcitvityInfo` 객체와 `FoxPerformanceContextInfo` 객체들을 DB에 기록하는 커스텀 로거를 작성해야 합니다. 상세한 예제는 [Fox Data Service Logging Sample](https://github.com/NeoDEEX/Samples/tree/master/WebService/DataService/Fox%20Data%20Service%20Logging%20Sample) 예제를 참고하십시오.

## Fox Biz/Data Service 성능 로그 및 DB Profile 제외 기능

`FoxDataRequest` 클래스와 `FoxBizRequest` 클래스의 `Diagnostic` 속성에 명시할 수 있는 플래그(`SupressPerfLogWrite`/`SupressDbProfileWrite`)가 추가 되었습니다. 이들 플래그를 명시함으로써 Fox Biz/Data Service에서 성능 로그 혹은 DB Profile을 로그에 기록하지 않도록 설정할 수 있습니다.

* `SupressPerfLogWrite` 플래그

  이 플래그가 명시된 `FoxBizRequest`/`FoxDataRequest`를 수신한 Fox Biz/Data Service는 성능 로그를 로거에 기록하지 않습니다. 불필요하고 의미 없는 성능 로그가 남지 않도록 호출 단위로 제어가 가능합니다.

* `SupressDbProfileWrite` 플래그

  이 플래그가 명시된 `FoxDataRequest`를 수신한 Fox Data Service는 DB Profile 로그를 로거에 기록하지 않습니다. 불필요하고 의미 없는 DB Profile 로그가 남지 않도록 호출 단위로 제어가 가능합니다.

  > 주: `FoxBizRequest` 클래스에는 이 플래그를 사용할 수 없습니다. DB Profile 기록 여부는 비즈 로직 메서드 내에서 제어를 해야 합니다.

> [Fox Data Service Logging Sample](https://github.com/NeoDEEX/Samples/blob/master/WebService/DataService/Fox%20Data%20Service%20Logging%20Sample/DataServiceClient/DbProfileViewForm.cs) 예제에서 이 `SupressPerfLogWrite` 플래그와 `SupressDbProfileWrite` 플래그를 사용하여 로그 조회 화면에서는 성능 로그나 DB Profile 로그가 남지 않도록 설정하고 있습니다.

## Fox Data Service : 타임아웃 설정 변경

Fox Data Service를 통해 수행되는 쿼리의 커맨드 타임아웃(command timeout)과 트랜잭션 타임아웃(transaction timeout)을 설정하는 방법이 변경되어 전역적으로 타임아웃을 제어할 수 있게 되었습니다.

* 4.5.4 버전 이전에는 데이터 서비스를 호출할 때 `FoxDataRequest` 객체의 `CommandTimeout` 속성에 값을 설정하지 않으면 디폴트 값(30초)이 적용되었습니다. 이러한 방식은 대부분의 시나리오에서 문제가 없지만 서버 측에서 전역적으로 디폴트 타임아웃을 늘이거나 줄이는 방법이 없었습니다.

* 4.5.4 버전에는 Fox Data Service의 서버 측에서 사용하는 커맨드 타임아웃과 트랜잭션 타임아웃을 지정하는 새로운 구성 설정이 추가 되었습니다.

  * `<dataService>` 섹션에 `commandTimeout` 속성과 `transactionTimeout` 속성을 추가하여 서버 측에서 사용하는 디폴트 타임아웃을 지정할 수 있습니다.

    ```xml
    <theone.configuration>
        <service>
        <dataService commandTimeout="120" transactionTimeout="180" />
        </service>
    </theone.configuration>
    ```

  * `<dataService>` 섹션의 구성 설정은 `FoxDataServiceConfig` 클래스의 `CommandTimeout`, `TransactionTimeout` 정적 속성을 통해 접근이 가능하며 수정도 가능합니다.

    ```cs
    FoxDataServiceConfig.CommandTimeout = 120;
    FoxDataServiceConfig.TransactionTimeout = 180;
    ```

  * 클라이언트가 `FoxDataRequest` 혹은 `FoxDataRequestCollection` 클래스에 `CommandTimeout` 속성 및 `TransactionTimeout` 속성을 설정하지 않으면 구성 설정에 지정된 타임아웃 설정값을 사용합니다.

    * 이를 지원하기 위해 이 두 속성의 타입이 `int` 에서 `int?` 타입으로 변경되었습니다.

    * `CommandTimeout` 속성의 경우, 구성 설정에 디폴트 타임아웃을 지정하지 않은 경우 `<connectionStrings>` 설정에서 지정된 연결 문자열의 커맨드 타임아웃이 사용됩니다.

    * `TransactionTimeout` 속성은 구성 설정에 디폴트 타임아웃을 지정하지 않은 경우 60초의 디폴트 값이 사용됩니다.

## Fox Biz Service : `IDictionary<string, object>` 타입 지원

Fox Biz Service를 사용할 때 비즈 메서드의 매개변수로 1개 이상의 `Dictionary<string, object>` 혹은 `FoxServiceParameterCollection` 타입을 사용할 수 있습니다. 상세한 내용은 [Collection Parameter Sample 예제](https://github.com/NeoDEEX/Samples/tree/master/WebService/BizService/Collection%20Parameter%20Sample)를 참고 하십시오.

```cs
// FoxBizRequest.Parameters 컬렉션에서 s, dic 키를 찾아서 매개변수 매핑
[FoxBizMethod]
public void ComplexTestMethod1(string s, IDictionary<string, object> dic)
{
    ......
}
```

## 뷰 제어 관련 새로운 API 추가

`FoxMenuViewModel` 클래스와 `FoxModuleLoader`에 뷰를 생성하고 로드하는데 사용할 수 있는 새로운 API들이 추가되었습니다.

* `FoxMenuViewModel.CreateViewModel` 메서드

    매개변수로 전달된 메뉴가 나타내는 뷰(`IFoxView`) 객체와 `FoxViewModel` 객체를 생성하여 반환합니다. 이 메서드 호출은 `DoViewOpen` 이벤트를 발생시키지 않기 때문에 생성된 뷰를 모달 다이얼로그와 같은 다양한 방법을 사용하여 표시할 수 있습니다. `CreateViewModel` 메서드는 권한 검사 및 뷰의 초기화 등 `SelectMenu` 메서드를 호출했을 때와 동일한 절차를 모두 거치게 됩니다.

* `FoxMenuViewModel.AttachViewModel` 메서드

    `CreateViewModel` 메서드 호출 등 기존에 생성된 뷰 객체를 `DoViewOpen` 이벤트를 통해 일반적인 뷰 생성 절차를 통해 화면에 표시하도록 합니다.

* `FoxMenuViewModel.CreateView` 메서드

    뷰 모델(`FoxViewModel`)에 연동되지 않는 뷰를 생성합니다. 뷰 모델과 연동되지 않기 때문에 `SelectMenu`, `CreateViewModel` 등의 메서드와 달리 이 메서드는 권한 검사나 초기화 과정을 거치지 않습니다.

* `FoxModuleLoader.CreateView` 메서드

    주어진 메뉴 정보를 사용하여 뷰를 생성하여 반환합니다. 기존에 프레임워크 내부에서만 사용되던 `internal` 메서드가 `public으로` 전환되었습니다.

기존의 Fox UI Model 기능에서 메뉴가 선택되었을 때 호출하는 메서드는 `SelectMenu`, `OpenMenu` 입니다. 이들 메서드는 뷰(view)를 즉시 표시하게 됩니다. 하지만 뷰 표시를 지연하여 표시하고자 하는 상황을 지원하는 API가 없었습니다.

예를 들어, 어플리케이션 시작시 여러 개의 뷰를 열고자 할 때 기존 `SelectMenu` 메서드를 사용하면 각 화면이 모두 열릴 때까지 어플리케이션은 잠기게 됩니다. 하지만 DevExpress의 `DocumentManager`가 제공하는 지연 로드(deferred loading)를 사용하고자 할 때 `SelectMenu` 혹은 `OpenMenu` 메서드로는 구현이 불가능했었습니다. 이제는 `CreateViewModel` 메서드를 사용하여 지연 로드의 구현을 손쉽게 수행할 수 있습니다. 구체적인 방법은 다음 예제를 참고하십시오.

[Deferred Load Example](https://github.com/NeoDEEX/Samples/tree/master/UIModel/Deferred%20Load%20Sample)

## FoxDbLoggerBase의 Fox Query 지원 강화

`FoxDbLoggerBase` 클래스에서 파생된 로거를 작성할 때 Fox Query를 위한 매개변수 설정을 위해 `GetParameterObject` 가상 메서드를 오버라이드할 수 있습니다. 이 메서드를 통해 Fox Query에서 필요한 매개변수를 다양한 방식(Entity, IDictionary, DataRow)으로 제공해 줄 수 있습니다.

[FoxDbLoggerBase를 사용한 DbProfile 기록 예제 보러 가기](https://github.com/NeoDEEX/Samples/blob/master/WebService/DataService/Fox%20Data%20Service%20Logging%20Sample/CommonLib/DbProfileDbLogger.cs)

## 버그 수정

* BUG 692: 다수 사용자 액세스 시 Updater에서 파일 공유 오류가 발생함.

  * 다수 사용자가 동시에 Updater를 통해 다운로드를 시도하는 경우 Updater 서버에서 오류가 발생할 수 있었던 문제가 해결되었습니다.

* BUG 695: Fox Data Service가 IIS 설치 디렉터리에서 foxml을 검색하여 Access Deny 오류가 발생함.

  * 연결 문자열에서 매퍼 설정이 존재하지 않는 경우 Working Directory에서 foxml 파일을 검색하던 문제를 AppDomain의 Base Directory를 검색하도록 수정하였습니다.

* BUG 707: [Security] Updater 서비스에 Pull path를 사용하면 해당 파일이 다운로드 됨.

  * Updater 서버에 다운로드 요청 시 상대 경로가 아닌 경우 400 Bad Request 오류가 반환되도록 수정되었습니다.

* BUG 712: Visual Studio에서 NeoDEEX 구성 파일을 변경하여도 변경 사항이 반영되지 않음.

  * Visual Studio는 파일 수정 후 저장 시 단순히 기존 파일을 업데이트하는 것이 아니라 기존 파일의 복사본을 수정하고 복사본 파일을 rename 하는 방식을 사용합니다. 이 때문에 FilesystemWatcher를 사용하는 구성 파일 변경 감시에 Rename을 추가하여 문제를 해결하였습니다.

* BUG 714: [Security] Updater 서비스에 상대 경로를 사용하면, 다운로드 기본 경로를 벗어나는 파일들을 다운로드 할 수 있음.

  * Updater 서버에서 파일 다운로드 요청 시 상대 경로를 사용하여 의도하지 않은 서버 측 파일을 다운로드 할 수 있었던 문제를 수정하였습니다.

  * 이 버그 수정 사항은 기존 설정으로 Updater 서버가 오류를 반환할 수 있습니다. 문제가 발생하는 경우 [Updater 설정 문제 해결](HotFix-4.5.3.3.md#문제-해결) 항목을 참고하여 기존 설정을 변경하십시오.

* Bug 725: FoxDataService의 `ExecuteMultiple` 메서드에서 디폴트 연결 문자열을 사용할 수 없음.

  * `ExecuteMultiple` 호출 시 단일 DB 연결을 사용하고자 할 때, 연결 문자열 중 디폴트 연결 문자열을 사용할 방법이 없었습니다. 이 문제를 해결하기 위해 `FoxDataRequestCollection` 클래스의 `DatabaseName` 속성에 `String.Empty` 혹은 빈 문자열("")을 할당하면 디폴트 연결 문자열을 사용하게 됩니다.

  * 이전 버전에서는 `String.Empty` 혹은 빈 문자열이 `DatabaseName` 속성에 할당되면 `FoxDataRequestCollection.DatabaseName` 속성을 사용하지 않고 컬렉션 내에 포함된 각 `FoxDataRequest` 객체의 `DatabaseName` 속성을 사용하여 데이터 베이스 연결을 수행하였었습니다.

---
