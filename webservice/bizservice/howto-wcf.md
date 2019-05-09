# How To : Fox Biz Service WCF 서비스 구성

이 문서는 Fox Biz Service(이하 비즈 서비스)를 위한 WCF 서비스를 작성하고 테스트를 위한 클라이언트 작성 방법을 단계적인 방법으로 설명합니다.

* [Fox Biz Service WCF 서비스 구성 예제 보러 가기](https://github.com/NeoDEEX/Samples/tree/master/WebService/BizService/Fox%20Biz%20Service%20WCF%20Demo)

* 목차

  * [웹 사이트 작성](#웹-사이트-작성)

  * [비즈 서비스 구성](#비즈-서비스-구성)

  * [비즈니스 클래스 작성](#비즈니스-클래스-작성)

  * [테스트용 클라이언트 작성](#테스트용-클라이언트-작성)

  * [문제 진단](#문제-진단)

  * [요약](#요약)

## 웹 사이트 작성

이 단계는 비즈 서비스를 제공할 새로운 웹 사이트를 작성하는 방법을 살펴 봅니다.

> 비즈 서비스는 기존의 웹 사이트에 필요한 어셈블리들의 참조를 추가하고 `web.config`를 수정하는 것만으로도 충분하며, 존재하는 임의의 웹 프로젝트(WebForm, MVC, Web API 등)에 추가하는 것도 가능합니다. 하지만 이 예제에서는 새로운 웹 사이트를 작성하는 것부터 시작할 것입니다.

웹 프로젝트를 작성하기 위해서는 닷넷 프레임워크 4.5 이상을 지원하는 Visual Studio 버전을 사용할 수 있습니다만 이 예제는 Visual Studio 2017을 사용할 것입니다.
다른 버전의 Visual Studio를 사용하더라도 웹 프로젝트의 생성 방법은 거의 동일 합니다.

1. New Project 메뉴를 선택하고 프로젝트 템플릿에서 ASP.NET Web Application 템플릿을 선택 합니다. 닷넷 프레임워크 버전은 4.5 이상이면 충분하지만 4.6 버전 이상을 사용하는 것이 좋습니다.

    ![프로젝트 템플릿 선택](images/howto-wcf-1.png "프로젝트 템플릿 선택")

2. ASP.NET Web Project 대화 상자에서 Empty 를 선택하고 확인을 클릭 합니다.

    * 웹 프로젝트에서 비즈 서비스 REST API 도 같이 제공하고자 한다면 Web API 옵션을 선택합니다.

    * 웹 프로젝트에서 HTML 컨텐츠를 제공하고자 한다면 Web Forms 혹은 MVC 옵션을 선택합니다.

    ![웹 어플리케이션 템플릿 선택](images/howto-wcf-2.png "웹 어플리케이션 템플릿 선택")

## 비즈 서비스 구성

WCF 비즈 서비스를 사용하기 위해서 필요한 어셈블리를 추가하고 WCF 서비스 구성 설정을 수행 합니다.

1. `TheOne`, `TheOne.Data`, `TheOne.ServiceModel`, `TheOne.ServiceModel.Activation` 어셈블리를 참조 추가 합니다.

    ![비즈 서비스 어셈블리 추가](images/howto-wcf-3.png "비즈 서비스 어셈블리 추가")

2. `Web.config` 구성 설정에 WCF 비즈 서비스를 추가 합니다.

    서비스 클래스로 `TheOne.ServiceModel.Biz.FoxBizService` 클래스를 사용하고, 상대 주소 값으로는 `.svc`로 종료되는 임의의 값을 사용할 수 있습니다. 이 예제는 `~/BizService.svc`을 사용하여 `http://xxxx/BizService.svc` 주소로 비즈 서비스를 사용할 수 있습니다.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <!-- 다른 구성 설정들 (생략) -->
      <system.serviceModel>
        <serviceHostingEnvironment>
          <serviceActivations>
            <add service="TheOne.ServiceModel.Biz.FoxBizService"
                relativeAddress="~/BizService.svc"/>
          </serviceActivations>
        </serviceHostingEnvironment>
      </system.serviceModel>
    </configuration>
    ```

    > 주) 위 서비스 설정은 디폴트 바인딩인 `BasicHttpBinding`을 사용합니다. 다른 바인딩을 사용하거나 WCF 동작(behaver) 설정이 필요하다면 `<services>` 요소에 `<service>` 요소를 추가하고 바인딩 및 서비스 동작 설정해야 합니다. 상세한 내용은 [WCF 서비스 구성](serverconfig.md#WCF-서비스-구성) 항목이나 MSDN 문서를 참고 하십시오.

3. 웹 브라우저를 구동하여 비즈 서비스의 주소를 입력하고 다음과 유사한 WCF 서비스 정보가 나타나는지 확인합니다. 이 화면이 정상적으로 나타나면 비즈 서비스 구성이 정상적으로 완료된 것입니다.

    ![비즈 서비스 구성 확인](images/howto-wcf-4.png "비즈 서비스 구성 확인")

## 비즈니스 클래스 작성

비즈 서비스는 서버 상에 존재하는 비즈니스 로직을 클라이언트가 호출할 수 있도록 해주는 WCF 서비스 입니다. 따라서 비즈니스 로직을 작성하는 것이 우선되어야 합니다.

비즈니스 로직을 작성하기 위해서는 임의의 `public` 클래스를 사용할 수 있습니다. 하지만 여러 데이터 원본(데이터베이스, 파일, 외부 서비스 등)을 액세스하고 그 결과들을 취합/변형 해야 하는 비즈니스 로직의 특성 상 분산/로컬 트랜잭션을 자동으로 관리해 주며, 손쉽게 성능 측정이 가능한 [Fox Transaction 기능](/transaction/README.md)을 사용하는 것이 편리합니다. 이 예제에서는 Fox Transaction이 제공하는 `FoxBizBase` 클래스로부터 파생된 비즈니스 로직 클래스를 작성할 것입니다.

1. 비즈니스 로직을 담기 위해 별도의 클래스 라이브러리 프로젝트를 솔루션에 추가 합니다.

    ![비즈니스 로직 프로젝트 추가](images/howto-wcf-5.png "비즈니스 로직 프로젝트 추가")
    ![비즈니스 로직 프로젝트 템플릿](images/howto-wcf-6.png "비즈니스 로직 프로젝트 템플릿")

2. `FoxBizClassAttribute`/`FoxBizMethodAttribute` 클래스 및 Fox Transaction을 사용하기 위해 `TheOne`, `TheOne.Data`, `TheOne.Transactions`, `TheOne.ServiceModel` 어셈블리를 참조 추가 합니다.

    ![NeoDEEX 참조 추가](images/howto-wcf-7.png "NeoDEEX 참조 추가")

3. 프로젝트 템플릿이 추가해 준 `Class1.cs` 파일을 제거 하고 `BizClass.cs`을 새로이 추가하고 다음과 같은 코드를 작성합니다. `FoxBizClassAttribute`와 `FoxBizMethodAttribute`를 사용하여 비즈 클래스와 비즈 메서드를 표시하고 있음에 주목하십시오.

    ```cs
    using TheOne.ServiceModel.Biz;
    using TheOne.Transactions;

    namespace BizModule
    {
        [FoxBizClass("데모비즈")]
        public class BizClass : FoxBizBase
        {
            [FoxBizMethod]
            public string Echo(string input)
            {
                return $"Hello Fox Biz Service: {input}";
            }
        }
    }
    ```

    > 주) `FoxBizClassAttribute` 클래스에는 비즈 클래스의 Id로 `데모비즈`라는 Id를 사용하고 있습니다. 명시적으로 `ClassId`를 제공하지 않는 경우, 네임스페이스를 포함하는 클래스의 전체 이름(이 경우 `BizModule.BizClass`)이 `ClassId`로 사용됩니다. 한편,  `FoxBizMethodAttribute` 클래스는 명시적으로 `MethodId`를 지정하지 않았기 때문에 메서드 이름(이 경우, `Echo`)이 `MethodId`로 사용됩니다.

4. 웹 프로젝트가 비즈니스 로직 모듈(프로젝트)를 사용할 수 있도록 프로젝트 참조를 추가 합니다.

    ![비즈니스 로직 프로젝트 참조](images/howto-wcf-8.png "비즈니스 로직 프로젝트 참조")

    > 주) 비즈 서비스에서 비즈니스 로직 모듈을 사용하기 위해서 반드시 참조를 추가할 필요는 없습니다. 비즈 서비스가 포함된 프로젝트(웹 프로젝트)가 비즈니스 로직 모듈(어셈블리)을 로드(Load)할 수 있는 위치(대부분 `bin` 디렉터리)에 DLL이 존재하기만 하면 됩니다. 다만 이 예제에서는 비즈니스 로직 모듈의 복사 과정을 단순화하기 위해 참조를 추가했을 뿐입니다.

5. 프로젝트에 새로운 XML 파일을 추가하여 NeoDEEX 구성 설정 파일을 생성 합니다. 파일 이름은 임의의 이름을 사용할 수 있지만 서버 측 구성 설정 파일은 .config 확장자를 갖는 것이 좋습니다. 이 예제는 `NeoDEEX.Server.config` 라는 이름을 사용하겠습니다. 또, NeoDEEX 구성 설정 파일을 사용하도록 `Web.config`에 다음 설정을 추가 합니다.

    ```xml
    <configuration>
      <!-- 다른 설정 생략 -->
      <appSettings>
        <add key="ConfigurationFileName" value="NeoDEEX.Server.config" />
      </appSettings>
      <!-- 다른 설정 생략 -->
    </configuration>
    ```

6. `NeoDEEX.Server.config` 파일을 편집하여 비즈 서비스가 로드할 비즈니스 로직 모듈을 추가 합니다.

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <theone.configuration  xmlns="http://schema.theonetech.co.kr/fx/config/2011/04/">
      <service>
        <bizService>
          <modules>
            <module name="BizModule" />
          </modules>
        </bizService>
      </service>
    </theone.configuration>
    ```

7. `Global.asax` 파일이 존재하지 않는 경우, 이 파일을 추가합니다.

    ![Global.asax 추가](images/howto-wcf-9.png "Global.asax 추가")

    이제 `Global.asax.cs` 에 다음 코드를 추가하여 비즈 서비스 구성을 수행합니다.

    ```cs
    public class Global : System.Web.HttpApplication
    {
        protected void Application_Start(object sender, EventArgs e)
        {
            // 비즈 서비스를 구성합니다.
            TheOne.ServiceModel.Biz.FoxBizServiceConfig.Configure();
        }
        // ... 이하 코드 생략 ...
    }
    ```

    이로써 비즈 서비스 구성이 완료 되었습니다. 간단한 비즈 서비스 구성은 이처럼 간단하지만 어플리케이션이 사용하는 WCF 바인딩이 BasicHttpBinding이 아니거나 [WCF 메시지 압축](../wcf/compress.md), [인증 설정](../wcf/authentication.md) 등이 추가되면 좀 더 복잡한 모습일 수도 있습니다.

    > 주) 기본적으로 비즈 서비스는 인증 확인을 수행합니다. 이는 권한이 없는 임의의 클라이언트가 비즈니스 로직을 호출할 수 없도록 하기 위함 입니다. 구성 설정을 통해 비즈 서비스 인증을 off 시킬 수 없습니다. 만약 인증을 사용하지 않도록 하려면 FoxBizService 클래스에서 파생된 클래스를 작성하고 FoxAuthenticationAttribute를 사용하거나 web.config 구성 설정을 사용하여 인증을 사용하지 않도록 구성해야 합니다 (권장하지 않음).

## 테스트용 클라이언트 작성

간단한 테스트를 위해 Console Application 클라이언트를 작성하는 방법을 설명합니다.

1. 기존 솔루션에 추가하거나 새로운 솔루션을 작성하여 Console Application 프로젝트를 생성합니다.

2. `TheOne`, `TheOne.SeviceModel`, `System.ServiceModel` 어셈블리를 참조 추가 합니다.

3. App.config 파일을 편집하여 NeoDEEX 구성 설정과 비즈 서비스 호출에 사용할 WCF 바인딩을 구성합니다.

    ```xml
    <configuration>
      <appSettings>
        <!-- NeoDEEX 구성 설정 파일 지정 -->
        <add key="ConfigurationFileName" value="NeoDEEX.Client.config"/>
      </appSettings>
      <system.serviceModel>
        <bindings>
          <basicHttpBinding>
            <!-- 바인딩 맵에서 참조할 WCF 바인딩 설정 -->
            <binding name="BasicHttpBinding">
              <security mode="None" />
            </binding>
          </basicHttpBinding>
        </bindings>
      </system.serviceModel>
    </configuration>
    ```

4. NeoDEEX 구성 설정을 위해 `NeoDEEX.Client.config` 파일을 추가하여 비즈 서비스의 [주소 맵(address map)](../wcf/addressmap.md)과 [바인딩 맵(binding map)](../wcf/bindingmap.md)을 구성 합니다.

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <theone.configuration  xmlns="http://schema.theonetech.co.kr/fx/config/2011/04/">
      <service defaultAddress="Default" defaultBindingMap="Default">
        <addresses>
          <address name="Default" baseUrl="http://localhost:49741" />
        </addresses>
        <bindingMaps>
          <bindingMap name="Default" bindingName="BasicHttpBinding"/>
        </bindingMaps>
      </service>
    </theone.configuration>
    ```

    NeoDEEX 구성 파일이 `.exe` 파일이 존재하는 디렉터리와 같은 디렉터리에 존재하도록 `NeoDEEX.Client.config` 구성 파일의 속성 창에서 `Copy to Output Directory`를 `Copy if newer`로 설정해 주어야 합니다.

    > 주) IIS Express를 사용하는 경우 주소 맵에서 `baseUrl` 속성를 명시할 때, 웹 사이트의 포트를 프로젝트에 맞게 변경해야 함에 유의 하십시오.

5. 다음과 같이 비즈 서비스를 호출하는 코드를 작성합니다. `FoxBizClient` 클래스의 인스턴스를 작성하고 `Execute` 메서드들을 호출하면 됩니다. 이때, 비즈 클래스의 Id(이 경우, `데모비즈`)와 비즈 메서드의 Id(이 경우, `Echo`)를 `FoxBizRequest` 객체의 생성자에 명시해야 합니다.

    `FoxBizRequest` 클래스의 `Parameters` 속성을 통해 매개변수를 전달한 것에 주목하십시오. `Parameters` 속성은 `IDictionary<string, object>` 인터페이스를 구현하는 `FoxServiceParameterCollection` 타입으로 설정된 키/값을 비즈 메서드의 매개변수로 전달합니다.

    ```cs
    using System;
    using TheOne.ServiceModel.Biz;

    namespace BizServiceClient
    {
        class Program
        {
            static void Main(string[] args)
            {
                using (var client = new FoxBizClient("BizService.svc"))
                {
                    var request = new FoxBizRequest("데모비즈", "Echo");
                    // 매개변수를 설정 합니다.
                    request.Parameters["input"] = "Wow!";

                    var response = client.Execute(request);
                    Console.WriteLine($"Result: {response.Result}");
                }
            }
        }
    }
    ```

6. 코드를 컴파일 하고 수행하여 비즈 서비스 호출이 정상적으로 수행되는지 확인합니다. 위 클라이언트 코드의 수행 결과는 다음과 같습니다.

    ```txt
    Result: Hello Fox Biz Service: Wow!
    ```

## 문제 진단

비스 서비스를 구성하고 호출하는데 문제가 발생하는 경우, 비즈 서비스의 진단 기능을 통해 문제의 원인을 파악하고 해결할 수 있습니다. 비스 서비스는 다양한 진단 기능을 제공합니다만 이 예제에서는 서비스 측 로그를 남기도록 하는 방법을 살펴보겠습니다.

1. 비스 서비스 프로젝트에서 `NeoDEEX.Server.config` 파일에 진단 설정을 다음과 같이 추가 합니다.

    ```xml
    <theone.configuration  xmlns="http://schema.theonetech.co.kr/fx/config/2011/04/">
      <service>
        <bizService>
          <!-- 비즈 서비스 로거 이름 명시-->
          <diagnostics loggerName="BizServiceLog" />
          ... 생략 ...
        </bizService>
      </service>
    </theone.configuration>
    ```

    이제 비즈 서비스는 `BizServiceLog`라는 이름의 로거(logger)를 사용하여 로그를 기록하게 됩니다.

2. 역시 `NeoDEEX.Server.config` 파일에 로거 설정을 다음과 같이 추가 합니다.

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <theone.configuration  xmlns="http://schema.theonetech.co.kr/fx/config/2011/04/">
      <service>
        ... 생략 ...
      </service>
      <logging>
        <loggers>
          <!-- 비즈 서비스를 위한 로거 설정 -->
          <logger name="BizServiceLog" filter="Verbose" providerType="TheOne.Diagnostics.Loggers.FoxTextFileLoggerProvider">
            <property name="FilePrefix" value="BizService" />
            <property name="Directory" value="C:\Temp"/>
          </logger>
        </loggers>
      </logging>
    </theone.configuration>
    ```

    `BizServiceLog` 로거 설정은 `C:\Temp` 디렉터리에 `BizService_YYYY-MM-DD.log` 파일을 생성하여 비즈 서비스의 로그를 기록하게 됩니다. 로깅 설정에 대한 상세한 내용은 [Fox Logging](/logging/README.md) 항목을 참고 하십시오.

3. 클라이언트를 구동하여 비즈 서비스를 호출하고 로그 파일의 내용을 살펴보십시오. 비즈 서비스는 `FoxBizServiceConfig.Configure` 메서드가 호출됨에 따라 초기화를 수행하고 다음과 같은 로그를 기록합니다.

    ```log
    FoxBizService: Start configuring ......
    starting to load predefined biz modules:
    Loading biz module: BizModule
        loaded assembly successfully: BizModule, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
    Loaded biz class: BizModule.BizClass, 1 biz methods, id=데모비즈
    FoxBizService: End configuring ......
    ```

    위 로그는 비즈 서비스가 `BizModule`을 로드했고 이 모듈 안에서 찾은 비즈 클래스의 목록을 보여 주고 있습니다. 초기화 로그로부터 비즈 모듈의 로드 여부 등 비스 서비스 구성이 정상적으로 수행되었는지 확인이 가능합니다.

    클라이언트가 비즈 서비스를 호출함에 따라 비즈 서비스는 다음과 비슷한 호출 로그도 기록하게 됩니다. 이 호출 로그로부터 비스 서비스가 어떤 비즈니스 로직을 수행하였고 수행하는데 소요된 시간 등을 파악할 수 있습니다. 호출 로그는 매 호출마다 고유한 로그 아이디(이 예제의 경우, `#00020:02958311#`)를 사용합니다. 다수의 클라이언트가 동시에 비즈 서비스를 호출하여 여러 호출에 대한 로그가 섞여서 기록되더라도 로그 아이디를 통해 어떤 호출의 로그 기록인가를 구분할 수 있습니다.

    ```log
    #00020:02958311#>> Biz Service Start: Execute  Requests=1  UserId=  ClientMachine=HOMEWORK  ClientIP=192.168.0.3;172.25.109.49  ClientMAC=60-45-CB-7F-91-D0;92-15-6E-E9-85-4C
    #00020:02958311#    Before Execute BizMethod: ClassId=데모비즈  MethodId=Echo  Type=BizModule.BizClass  Method=Echo  in-params=1
    #00020:02958311#    After Invoke: elpased = 10.0 msec, out-params:0
    #00020:02958311#>> Biz Service End: Execute  elapsed = 30.8 msec
    ```

    > 주) 위에서 표시된 로그 메시지는 설명을 위해 일부 정보는 제외하여 표시하였습니다. 실제 로그 파일에는 로그 수준(`Verbose`, `Information`, `Warning`, `Error` 등)과 로그 발생 시간, 로그 원본(source) 등의 정보가 포함되어 있습니다.

## 요약

비즈 서비스는 NeoDEEX 프레임워크 내에 WCF 서비스를 구현하여 제공함으로써 간단한 구성 설정 만으로 서버 상에 존재하는 비즈니스 로직 코드를 수행할 수 있는 환경을 구성할 수 있습니다.

웹 사이트 작성과 비즈 서비스 구성은 어플리케이션 작성 시 최초 1회만 수행하면 됩니다. 비즈 서비스 구성이 완료되면 비즈니스 로직을 작성하여 추가하기만 하면 됩니다. 모든 개발자가 웹 사이트를 구성할 필요도 없으며 비즈 서비스 구성을 어떻게 해야 하는지 알 필요도 없습니다.

비즈 서비스를 사용하면 단순히 비즈니스 로직 코드를 작성하는 것만으로 클라이언트가 손쉽게 비즈니스 로직을 호출할 수 있도록 해줍니다.

---