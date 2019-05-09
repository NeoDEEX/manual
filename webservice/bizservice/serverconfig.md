# Fox Biz Service 서버 구성

Fox Biz Service(이하 비즈 서비스)는 WCF 혹은 REST API를 사용하여 서버를 구성할 수 있습니다.

> 비즈 서비스는 기본적으로 WCF를 사용하도록 설계/구현되어 있으므로 별도의 설명없이 비즈 서비스라고 지칭하면 WCF를 사용하는 서비스를 말합니다. 한편 HTTP/JSON을 사용하는 RESTful 서비스는 비즈 서비스 REST API라고 부릅니다.

이 문서는 WCF와 REST API를 통해 비즈 서비스와 비즈 서비스 REST API를 구성하는 방법에 대해 설명합니다. 비즈 서비스는 [데이터 서비스(Fox Data Service)](../dataservice/README.md)와 달리 웹 서버 설정 외에도 추가적인 구성 및 코드 작성이 필요합니다. 비즈 서비스가 호출할 비즈니스 로직 클래스/메서드들에 대한 정보가 필요하기 때문입니다. 비즈니스 로직 클래스/메서드에 대한 구성은 이들 클래스가 포함된 비즈 모듈(어셈블리)를 명시하는 NeoDEEX 구성 설정과 Attribute를 통해 클래스에 비즈니스 로직 클래스임을 명시하는 과정이 포함됩니다.

목차

* [WCF 구성](#WCF-구성)

* [REST API 구성](#REST-API-구성)

* [비즈니스 로직 모듈 구성](#비즈니스-로직-모듈-구성)

* [요약](#요약)

## WCF 구성

WCF를 사용하여 서버를 구축하는 방법은 여러 가지 방법이 있을 수 있습니다. WCF 자체가 웹 사이트(IIS)에서 호스팅 되거나 독립 호스트(`.exe`)로서 WCF 서비스를 제공할 수 있기 때문입니다. 이 문서는 가장 많이 사용되는 웹 사이트에서 WCF 비즈 서비스를 구성하는 방법에 대해 살펴보도록 하겠습니다. 독립 호스트를 사용하는 방법은 다른 WCF 문서들을 참고하시길 바랍니다.

### WCF 웹 사이트 기본 구성

비즈 서비스의 WCF 서비스를 웹 사이트에서 제공하기 위해서는 먼저 웹 사이트가 준비되어야 합니다. Visual Studio를 사용하여 새로운 웹 프로젝트를 작성하거나 기존 웹 프로젝트를 사용할 수 있으며, 이미 WCF를 사용 중인 기존 웹 사이트를 이용할 수도 있습니다.

웹 사이트는 WCF에 필수적인 어셈블리들(`System.ServiceModel`, `System.ServiceModel.Activation` 등)이 참조되어 있어야 합니다. 비즈 서비스를 사용하기 위해서는 `TheOne.ServiceModel` 과 `TheOne.ServiceModel.Activation` 어셈블리가 참조되어야 합니다. 또, 이 두 어셈블리가 의존성을 갖는 `TheOne`, `TheOne.Data` 어셈블리도 참조가 필요합니다. 그리고 [Fox Transaction 기능](/transaction/README.md)을 비즈니스 로직 계층으로 사용하고자 한다면 `TheOne.Transactions` 어셈블리도 참조해야 합니다.

### WCF 서비스 구성

비즈 서비스는 WCF 서비스를 위해 `IFoxBizService` 인터페이스와 `FoxBizService` 클래스를 기본적으로 제공합니다. 이 두 클래스를 사용하여 다음과 같이 `web.config` 구성 설정을 추가하여 비즈 서비스의 WCF 서비스를 구성할 수 있습니다.

> WCF 비즈 서비스를 작성하는 구체적인 예제는 [How To : Fox Biz Service WCF 서비스 구성](howto-wcf.md) 항목을 참고 하십시오.

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

위와 같은 설정은 IIS(혹은 IIS Express)에 호스팅되는 WCF 서비스의 기본 설정을 따르게 됩니다. 즉, 바인딩으로 `BasicHttpBinding`을 사용하고 WSDL에 대한 HTTP GET을 허용하지 않는 등의 디폴트 설정이 사용됩니다. 따라서 `BasicHttpBinding`이 아닌 다른 바인딩을 사용하거나 WCF 서비스 동작(behavior)를 적용하기 위해서는 WCF 서비스 구성의 표준 방법을 사용해야 합니다. 다음은 `WsHttpBinding`을 사용하고 WSDL에 대한 HTTP GET을 지원하는 WCF 서비스 구성 방법을 보여줍니다.

```xml
<configuration>
  <!-- 다른 구성 설정들 (생략) -->
  <appSettings>
    <add key="ConfigurationFileName" value="NeoDEEX.Server.config" />
  </appSettings>
  <system.serviceModel>
    <serviceHostingEnvironment>
      <serviceActivations>
        <add service="TheOne.ServiceModel.Biz.FoxBizService"
             relativeAddress="~/BizService.svc" />
      </serviceActivations>
    </serviceHostingEnvironment>
    <services>
      <service name="FoxBizService" behaviorConfiguration="DefaultBehavior" />
    </services>
    <behaviors>
      <serviceBehaviors>
        <behavior name="DefaultBehavior">
          <serviceMetadata httpGetEnabled="true" />
        </behavior>
      </serviceBehaviors>
    </behaviors>
  </system.serviceModel>
</configuration>
```

> 주) `<service>` 항목의 `name` 속성이 클래스 이름이 아닌 것에 주의 하십시오. 비즈 서비스의 WCF 서비스 구성 설정은 클래스 이름이 아닌 `FoxBizService`라는 구성 설정 이름이 사용됩니다.

NeoDEEX는 WCF 서비스들을 위해 표준적인 방법으로 [바인딩 맵(binding map)](../wcf/bindingmap.md)을 지원합니다. 바인딩 맵은 다수의 WCF 서비스가 공통으로 사용하는 설정을 기본값으로 제공하여 `web.config`에서 WCF 서비스에 관련된 구성 설정을 크게 줄여 줍니다.

바인딩 맵을 사용하기 위해서는 [FoxServiceHostFactory 클래스](../wcf/servicefactory.md)를 사용해야 합니다. 일반적인 WCF 서비스는 `.svc` 파일을 통해 `FoxServiceHostFactory` 적용이 가능하지만 `web.config` 만을 사용하는 경우 `<serviceActivations/service>` 요소에 `factory` 속성을 추가해야 합니다. 다음은 `FoxServiceHostFactory`를 적용한 `web.config` 구성 설정 예를 보여줍니다.

```xml
<configuration>
  <!-- 다른 구성 설정들 (생략) -->
  <appSettings>
    <!-- NeoDEEX 외부 구성 파일 설정 -->
    <add key="ConfigurationFileName" value="NeoDEEX.Server.config" />
  </appSettings>
  <system.serviceModel>
    <serviceHostingEnvironment>
      <serviceActivations>
        <add service="TheOne.ServiceModel.Biz.FoxBizService"
             relativeAddress="~/BizService.svc"
             factory="TheOne.ServiceModel.Activation.FoxServiceHostFactory"/>
      </serviceActivations>
    </serviceHostingEnvironment>
    <customBinding>
      <binding name="DefaultBinding">
        <gzipMessageEncoding innerMessageEncoding="textMessageEncoding" mode="enabled"/>
        <httpTransport/>
      </binding>
    </customBinding>
    <behaviors>
      <serviceBehaviors>
        <behavior name="DefaultBehavior">
          <serviceDebug includeExceptionDetailInFaults="true"/>
          <serviceMetadata httpGetEnabled="true" />
        </behavior>
      </serviceBehaviors>
    </behaviors>
  </system.serviceModel>
</configuration>
```

다음은 NeoDEEX 구성 설정(`neodeex.server.config`) 파일에 바인딩 맵의 정의를 보여주고 있습니다.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<theone.configuration  xmlns="http://schema.theonetech.co.kr/fx/config/2011/04/">
  <service defaultBindingMap="Default">
    <bindingMaps>
      <bindingMap name="Default" bindingName="DefaultBinding" serviceBehavior="DefaultBehavior" />
    </bindingMaps>
  </service>
</theone.configuration>
```

## REST API 구성

비즈 서비스의 REST API는 HTTP와 JSON을 사용하는 RESTful 서비스를 통해 비즈 서비스의 기능을
사용할 수 있도록 해줍니다. 비즈 서비스의 REST API는 ASP.NET Web API 2를 사용하여 구현되었습니다. 따라서 REST API를 사용하기 위해서는 Web API 구성이 선행되어야 합니다.

> 비즈 서비스 REST API를 작성하는 구체적인 예제는 [How To : Fox Biz Service REST API 서비스 구성](howto-rest.md) 항목을 참고 하십시오.

### Web API 웹 사이트 기본 구성

웹 사이트가 Web API를 지원하도록 하기 위해서는 Web API에 관련된 몇 가지 어셈블리(`System.Net.Http.Formatting`, `System.Net.Http`, `System.Net.Web.WebHost` 등)를 참조해야 하며 `Global.asax`, `Global.asax.cs`, `WebApiConfig.cs` 등의 코드가 준비되어야 하고, 필요하다면 `web.config`를 변경해야 할 수도 있습니다.

Web API 구성은 Visual Studio에서 웹 프로젝트를 처음 생성할 때 Web API를 지원하도록 구성하는 것이 가장 쉽습니다. 만약 이미 웹 프로젝트가 만들어져 있다면, 프로젝트의 새 항목 추가에서 `스캐폴드 추가`에서 Web API 지원을 추가할 수 있습니다.

Web API 구성이 완료되어 있다면 비즈 서비스에 관련된 어셈블리들을 참조해야 합니다.
비즈 서비스를 사용하기 위해서는 기본적으로 `TheOne`, `TheOne.Data`, `TheOne.ServiceModel`과 `TheOne.ServiceModel.Activation` 어셈블리가 참조되어야 합니다. 또, REST API를 위해 `NeoDEEX.ServiceModel.Web`, `NeoDEEX.ServiceModel.Web.Http` 어셈블리 역시 참조 해야 합니다. 그리고 [트랜잭션 컴포넌트(Fox Transaction)](/transaction/README.md)와 관련하여 필요하다면 `TheOne.Transactions` 등의 어셈블리 참조가 필요할 수도 있습니다.

### REST API 구성 설정

Web API는 WCF와 달리 `web.config`를 통한 구성 설정보다는 코드에 의한 구성 설정을 사용합니다. 가장 핵심적인 구성 설정은 URL 라우팅에 대한 설정 입니다. 비즈 서비스의 REST API를 어떤 URL으로 노출할 것인가를 `WebApiConfig.cs` 파일을 수정하여 추가해야 합니다. 다음은 라우팅 설정의 예를 보여 줍니다.

```cs
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        config.MapHttpAttributeRoutes();

        config.Routes.MapHttpRoute(
            name: "FoxBizServiceRoute",
            routeTemplate: "api/bizservice/{action}",
            defaults: new { controller = "FoxBizService" }
        );
    }
}
```

## 비즈니스 로직 모듈 구성

클라이언트가 요청한 비즈니스 로직 클래스/메서드를 호출하기 위해서 비즈 메서드는 어떤 모듈(어셈블리)에 어떤 비즈니스 로직 클래스가 존재하는지 파악할 수 있어야 합니다. NeoDEEX는 비즈니스 로직 클래스를 나타내기 위해 `FoxBizClassAttribute` 클래스를, 비즈니스 로직 메서드를 나타내기 위해 `FoxBizMethodAttribute` 클래스를 사용합니다.

```cs
namespace Namespace
{
    [FoxBizClass]
    public class Class1
    {
        [FoxBizMethod]
        public DataSet Method1()
        {
            // ... 생략...  
        }

        [FoxBizMethod]
        public string Method2(int param1, string param2)
        {
            // ... 생략...  
        }
    }
}
```

비즈 서비스는 NeoDEEX 구성 설정(`neodeex.server.config`)의 `<service/bizService>` 요소에서 나열된 비즈니스 로직 모듈들에서 비즈니스 로직 클래스들을 찾습니다. 다음 구성 설정은 `MyBizModule.dll` 내에 포함된 `public` 클래스들 중 `FoxBizClassAttribute`가 설정된 모든 클래스들을 비즈니스 로직 클래스로써 사용하게 됩니다.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<theone.configuration  xmlns="http://schema.theonetech.co.kr/fx/config/2011/04/">
  <service>
    <bizService>
      <modules>
        <module name="MyBizModule" />
      </modules>
    </bizService>
  </service>
</theone.configuration>
```

비즈니스 로직 모듈(들)에서 비즈니스 로직 클래스를 찾는 과정은 `FoxBizServiceConfig` 클래스의 `Configure` 메서드가 호출될 때 수행하게 됩니다. 따라서 서버 프로세스 구동 시 `FoxBizServiceConfig.Configure` 메서드를 호출해 주어야 합니다. 다음은 `Global.asax.cs` 파일에서 비즈니스 로직 모듈을 로드하고 구성하도록 하는 예제 코드를 보여 줍니다.

```cs
public class Global : HttpApplication
{
    void Application_Start(object sender, EventArgs e)
    {
        // Fox Biz Service 구성
        TheOne.ServiceModel.Biz.FoxBizServiceConfig.Configure();

        // ... 다른 초기화 코드들 ...
    }
}
```

## 요약

비즈 서비스는 비즈니스 로직이 WCF 혹은 REST API를 통해 외부에 손쉽게 노출될 수 있도록 해줍니다. 비즈 서비스를 WCF 혹은 REST API로 제공하기 위해 필요한 것은 간단한 서버 측 구성 설정과 몇 줄의 코드 뿐입니다.

---