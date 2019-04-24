# Fox Data Service 서버 구성

Fox Data Service(이하 데이터 서비스)는 WCF 혹은 REST API를 사용하여 서버를 구성할 수 있습니다. 데이터 서비스는 기본적으로 WCF를 사용하도록 설계/구현되어 있으므로 별도의 설명없이 데이터 서비스라고 지칭하면 WCF를 사용하는 서비스를 말합니다. 한편 HTTP/JSON을 사용하는 RESTful 서비스는 데이터 서비스 REST API라고 부릅니다.

이 문서는 WCF와 REST API를 통해 데이터 서비스와 데이터 서비스 REST API를 구성하는 방법에 대해 개략적으로 설명합니다. 구체적이고 단계적인 방법은 다음 문서를 참고 하십시오.

* [How To : Fox Data Service WCF 서비스 구성](howto-wcf.md)
* [How To : Fox Data Service REST API 서비스 구성](howto-rest.md)

## WCF 구성

WCF를 사용하여 서버를 구축하는 방법은 여러 가지 방법이 있을 수 있습니다. WCF 자체가 웹 서버(IIS)에서 호스팅 되거나 독립 호스트(.exe)로서 WCF 서비스를 제공할 수 있기 때문입니다. 이 문서는 가장 많이 사용되는 웹 서버에서 데이터 서비스의 WCF 서비스를 구성하는 방법에 대해 살펴보도록 하겠습니다. 독립 호스트를 사용하는 방법은 다른 WCF 문서들을 참고하시길 바랍니다.

WCF를 사용하는 데이터 서비스를 작성하는 구체적인 예제는 [How To : WCF 서비스 구성](howto-wcf.md) 항목을 참고 하십시오.

### WCF 웹 사이트 기본 구성

데이터 서비스의 WCF 서비스를 웹 서버에서 제공하기 위해서는 먼저 웹 사이트가 준비되어야 합니다. Visual Studio를 사용하여 새로운 웹 프로젝트를 작성하거나 기존 웹 프로젝트를 사용할 수 있으며, 이미 WCF를 사용 중인 기존 웹 사이트를 이용할 수도 있습니다.

웹 사이트는 WCF에 필수적인 어셈블리들(`System.ServiceModel`, `System.ServiceModel.Activation` 등)이 참조되어 있어야 합니다. 데이터 서비스를 사용하기 위해서는 `TheOne.ServiceModel` 과 `TheOne.ServiceModel.Activation` 어셈블리가 참조되어야 합니다. 또, 이 두 어셈블리가 의존성을 갖는 `TheOne`, `TheOne.Data` 어셈블리도 참조가 필요합니다. 그리고 데이터베이스 액세스와 관련하여 필요하다면 `TheOne.Data.Odp` 혹은 `TheOne.Data.OracleClient` 등의 어셈블리 참조가 필요할 수도 있습니다.

### WCF 서비스 구성

데이터 서비스는 WCF 서비스를 위해 `IFoxDataService` 인터페이스와 `FoxDataService` 클래스를 제공합니다.
이 두 클래스를 사용하여 다음과 같이 `web.config` 구성 설정을 추가하는 것으로만 데이터 서비스의 WCF 서비스를 구성할 수 있습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <!-- 다른 구성 설정들 (생략) -->
  <system.serviceModel>
    <serviceHostingEnvironment>
      <serviceActivations>
        <add service="TheOne.ServiceModel.Data.FoxDataService"
             relativeAddress="~/DataService.svc"/>
      </serviceActivations>
    </serviceHostingEnvironment>
  </system.serviceModel>
</configuration>
```

위와 같은 설정은 IIS(혹은 IIS Express)에 호스팅되는 WCF 서비스의 기본 설정을 따르게 됩니다. 즉, 바인딩으로 `BasicHttpBinding`을 사용하고 WSDL에 대한 HTTP GET을 허용하지 않는 등의 디폴트 설정이 사용됩니다. 따라서 `BasicHttpBinding`이 아닌 다른 바인딩을 사용하거나 WCF 서비스 Behavior를 적용하기 위해서는 WCF 서비스 구성의 표준 방법을 사용해야 합니다. 다음은 `WsHttpBinding`을 사용하고 WSDL에 대한 HTTP GET을 지원하는 WCF 서비스 구성 방법을 보여줍니다.

```xml
<configuration>
  <!-- 다른 구성 설정들 (생략) -->
  <appSettings>
    <add key="ConfigurationFileName" value="NeoDEEX.Server.config" />
  </appSettings>
  <system.serviceModel>
    <serviceHostingEnvironment>
      <serviceActivations>
        <add service="TheOne.ServiceModel.Data.FoxDataService"
             relativeAddress="~/DataService.svc" />
      </serviceActivations>
    </serviceHostingEnvironment>
    <services>
      <service name="FoxDataService" behaviorConfiguration="DefaultBehavior" />
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

>주) `<service>` 항목의 `name` 속성이 클래스 이름이 아닌 것에 주의 하십시오. 데이터 서비스의 WCF 서비스 구성 설정은 클래스 이름이 아닌 `FoxDataService`라는 구성 설정 이름이 사용됩니다.

NeoDEEX는 WCF 서비스들을 위해 표준적인 방법으로 바인딩 맵(binding map)을 지원합니다. 바인딩 맵은 다수의 WCF 서비스가 공통으로 사용하는 설정을 기본값으로 제공하여 `web.config`에서 WCF 서비스에 관련된 구성 설정을 크게 줄여 줍니다.

바인딩 맵을 사용하기 위해서는 `FoxServiceHostFactory` 클래스를 사용해야 합니다. 일반적인 WCF 서비스는 .svc 파일을 통해 `FoxServiceHostFactory` 적용이 가능하지만 `web.config` 만을 사용하는 경우 `<serviceActivations/service>` 요소에 `factory` 속성을 추가해야 합니다. 다음은 `FoxServiceHostFactory`를 적용한 구성 설정 예를 보여줍니다.

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
        <add service="TheOne.ServiceModel.Data.FoxDataService"
             relativeAddress="~/DataService.svc"
             factory="TheOne.ServiceModel.Activation.FoxServiceHostFactory"/>
      </serviceActivations>
    </serviceHostingEnvironment>
    <customBinding>
      <binding name="DefaultBinding">
        <gzipMessageEncoding innerMessageEncoding="textMessageEncoding" mode="disabled"/>
        <httpTransport maxReceivedMessageSize="10000000" transferMode="Streamed" keepAliveEnabled="false"/>
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

>주) 닷넷 표준 구성 파일의 제약으로 인해 `<serviceActivations>` 항목을 통해 서비스를 구성하는 경우, `factory` 속성에는 `ServiceHostFactory`에서 파생된 클래스 이름만을 넣을 수 있습니다. 따라서 기본 바인딩 맵(`defaultBindingMap`)만이 데이터 서비스에 적용됩니다. 기본 바인딩 맵이 아닌 다른 바인딩 맵을 사용해야 하는 경우에는 `<serviceActivations>` 대신 전통적인 방법대로 .svc 파일을 만들고 `@ServiceHost` 디렉티브의 `Factory` 속성에 `FoxServiceHostFactory` 클래스 이름과 바인딩 맵 이름을 ; 문자로 구분하여 표시하면 됩니다.
>
  > <%@ServiceHost Service="TheOne.ServiceModel.Data.FoxDataService" Factory="TheOne.ServiceModel.Activation.FoxServiceHostFactory;MyBindingMap" %>
>
>**바인딩 맵이 다수의 WCF 구성 설정을 간편하게 하기 위한 목적이지만 데이터 서비스는 단일 WCF 서비스를 사용하기 때문에 바인딩 맵을 사용하려고 노력할 필요는 없습니다.**

## REST API 구성

데이터 서비스의 REST API는 HTTP와 JSON을 사용하는 RESTful 서비스를 통해 데이터 서비스의 기능을 사용할 수 있도록 해줍니다. 데이터 서비스의 REST API는 ASP.NET Web API 2를 사용하여 구현되었습니다. 따라서 REST API를 사용하기 위해서는 Web API 구성이 선행되어야 합니다.

데이터 서비스 REST API를 작성하는 구체적인 예제는 [How To : REST API 구성](howto-rest.md) 항목을 참고 하십시오.

### Web API 웹 사이트 기본 구성

웹 사이트가 Web API를 지원하도록 하기 위해서는 Web API에 관련된 몇 가지 어셈블리(`System.Net.Http.Formatting`, `System.Net.Http`, `System.Net.Web.WebHost` 등)를 NuGet을 통해 참조해야 하며 `Global.asax`, `Global.asax.cs`, `WebApiConfig.cs` 등 의 코드가 준비되어야 하고, 필요하다면 `web.config`를 변경해야 할 수도 있습니다.

Web API 구성은 Visual Studio에서 웹 프로젝트를 처음 생성할 때 Web API를 지원하도록 구성하는 것이 가장 쉽습니다. 만약 이미 웹 프로젝트가 만들어져 있다면, 프로젝트의 새 항목 추가에서 '스캐폴드 추가'에서 Web API 지원을 추가할 수 있습니다.

Web API 구성이 완료되어 있다면 데이터 서비스에 관련된 어셈블리들을 참조해야 합니다. 데이터 서비스를 사용하기 위해서는 기본적으로 `TheOne`, `TheOne.Data`, `TheOne.ServiceModel`과 `TheOne.ServiceModel.Activation` 어셈블리가 참조되어야 합니다. 또, REST API를 위해 `NeoDEEX.ServiceModel.Web`, `NeoDEEX.ServiceModel.Web.Http` 어셈블리 역시 참조 해야 합니다. 그리고 데이터베이스 액세스와 관련하여 필요하다면 `TheOne.Data.Odp` 혹은 `TheOne.Data.OracleClient` 등의 어셈블리 참조가 필요할 수도 있습니다.

### REST API 구성 설정

Web API는 WCF와 달리 `web.config`를 통한 구성 설정보다는 코드에 의한 구성 설정을 사용합니다. 가장 핵심적인 구성 설정은 URL 라우팅에 대한 설정 입니다. 데이터 서비스 REST API를 어떤 URL으로 노출할 것인가를 `WebApiConfig.cs` 파일을 수정하여 추가해야 합니다. 다음은 라우팅 설정의 예를 보여 줍니다.

```csharp
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        config.MapHttpAttributeRoutes();
        // Fox Data Service 라우팅 설정
        config.Routes.MapHttpRoute(
            name: "FoxDataServiceRoute",
            routeTemplate: "api/dataservice/{action}",
            defaults: new { controller = "FoxDataService" }
        );
    }
}
```

## Summary

데이터 서비스는 비즈니스 로직을 갖고 있지 않고 단순히 쿼리를 수행하는 서버 측 기능 작성을 손쉽게 해줍니다. 데이터 서비스를 WCF 혹은 REST API로 제공하기 위해 필요한 것은 간단한 서버 측 구성 설정과 몇 줄의 코드 뿐입니다.

---