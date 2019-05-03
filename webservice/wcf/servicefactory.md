# 서비스 호스트 팩터리(Service Host Factory)

Fox Web Services는 `FoxServiceHostFactory` 클래스를 통해 서버 측에서 WCF 서비스를 생성하고 구성할 때 [바인딩 맵](bindingmap.md)을 사용하여 서비스 구성에 필요한 구성 설정을 크게 줄여 줍니다. `FoxServiceHostFactory` 클래스는 WCF에서 제공하는 `ServiceHostFactory`에서 파생된 서비스 호스트 팩터리로서 `TheOne.ServiceModel.Activation` 어셈블리에 포함되어 있습니다.

## 서비스 팩터리 사용

`FoxServiceHostFacoty`를 사용하기 위해서는 표준적인 방법으로 `.svc` 파일의 `ServiceHost` 지시자(directive)의 `Factory` 속성을 통해 사용할 수 있습니다. 다음은 `ServiceHost` 지시자에서 `FoxServiceHostFactory` 클래스를 `Factory` 속성에 설정하는 예를 보여주고 있습니다.

```aspx
<%@ ServiceHost Language="C#" Debug="true"
    Service="WcfServiceWeb.WcfService" CodeBehind="WcfService.svc.cs"
    Factory="TheOne.ServiceModel.Activation.FoxServiceHostFactory" %>
```

위와 같은 설정에서 `FoxServiceHostFactory`는 [바인딩 맵](bindingmap.md)에서 설정된 디폴트 바인딩 맵 항목을 사용하여 바인딩과 서비스 동작을 자동으로 설정합니다. 따라서 `web.config`에서 이 서비스를 위한 `<endpoint>` 설정을 구성하지 않아도 됩니다.

`.svc` 파일을 사용하지 않고 `web.config`에서 `<serviceActivations>` 요소를 사용하여 WCF 서비스를 구성할 때에서도 `Factory` 속성에 설정이 가능합니다. `<serviceActivations>` 요소를 사용하여 WCF 서비스를 작성하는 방법은 `.svc` 파일을 작성하지 않아도 된다는 장점이 있지만 기업용 어플리케이션에서 수백개에 달하는 대량의 WCF 서비스를 작성할 때에서는 `web.config`에 구성 설정이 너무 많아 진다는 단점을 가지므로 권장되지 않습니다. 소수의 WCF 서비스들에 대해서만 `<serviceActivations>` 요소를 적용하기를 권장합니다. 다음은 `<serviceActivation>` 요소를 통해 WCF 서비스를 구성할 때 `FoxServiceHostFactory`를 적용하는 예를 보여주고 있습니다.

```xml
<serviceHostingEnvironment aspNetCompatibilityEnabled="true"
  multipleSiteBindingsEnabled="true">
  <serviceActivations>
    <add service="WcfServiceWeb.WcfService" relativeAddress="WcfService1.svc"
          factory="TheOne.ServiceModel.Activation.FoxServiceHostFactory"/>
  </serviceActivations>
</serviceHostingEnvironment>
```

`FoxServiceHostFactory`를 사용하면 `바인딩 맵`을 사용하기 때문에 구성이 간편하여 대량의 WCF 구성에 용이하다는 장점이 있지만 제약 사항을 가지고 있습니다. `FoxServiceHostFactory`는 서비스 인터페이스를 하나만 구현하는 WCF 서비스만을 지원합니다. 즉 서비스 클래스가 하나의 서비스 인터페이스만을 사용해야 한다는 것입니다.

`FoxServiceHostFacotry`는 서비스 클래스에서 서비스 인터페이스를 찾고 서비스 인터페이스가 1개인 경우에만 하나의 바인딩 맵을 적용합니다. 만약 서비스 클래스가 2개 이상의 서비스 인터페이스를 사용하는 경우, `FoxServiceHostFactory`는 web.config에서 `<endpoint>` 설정을 찾아 구성하는 기본 서비스 팩터리(ServiceHostFactory)와 동일하게 작동합니다. 즉, `web.config`에서 WCF 서비스를 위해 `<endpoint>` 요소를 정의해 주어야 합니다.

하지만 [전통적인 WCF 서비스 구성의 해결 방법](problems.md#해결-방법)에서 언급한대로 대부분의 WCF 서비스는 하나의 서비스 인터페이스만을 가지고 있기 때문에 `FoxServiceHostFactory`의 제약 사항은 큰 문제가 되지 않습니다. 2개 이상의 서비스 인터페이스를 사용하는 몇 개의 WCF 서비스에 대해 `<endpoint>` 설정을 유지하는 것은 개발 및 유지보수를 하는데 큰 문제가 되지 않기 때문입니다.

## 바인딩 맵 명시

`FoxServiceHostFactory`는 기본적으로 [바인딩 맵](bindingmap.md)에서 지정된 디폴트 바인딩 맵 항목을 사용하여 바인딩과 서비스 동작(service behavior)을 구성합니다. 대부분의 서비스들이 디폴트 바인딩 맵에서 지정된 바인딩과 서비스 동작을 사용하겠지만 항상 그러한 것은 아닙니다. 만약 어떤 서비스가 디폴트 바인딩 맵이 아닌 다른 바인딩 맵을 사용해야 한다면 이것을 `FoxServiceHostFactory`에게 알려주어야 합니다.

디폴트 바인딩 맵이 아닌 다른 바인딩 맵 항목을 사용해야 한다면 `.svc` 파일의 `ServiceHost` 지시자(directive)의 `Service` 속성을 통해 사용할 수 있습니다.

> 원래  `Service` 속성은 WCF 서비스를 구현하는 서비스 클래스의 이름을 명시하는데 사용됩니다.

서비스 팩터리로 `FoxServiceHostFactory`가 사용되면 서비스 클래스 이름 뒤에 세미콜론(;) 문자로 구분되어 바인딩 맵 이름을 추가로 명시할 수 있습니다. 다음은 바인딩 맵 이름이 명시된 `ServiceHost` 지시자의 예를 보여주고 있습니다.

```aspx
<%@ ServiceHost Language="C#" Debug="true"
    Service="WcfServiceWeb.WcfService3;netHttp" CodeBehind="WcfService3.svc.cs"
    Factory="TheOne.ServiceModel.Activation.FoxServiceHostFactory" %>
```

위와 같은 설정에서 `FoxServiceHostFactory`는 바인딩 맵에서 이름이 `netHttp`인 항목을 찾아 그 항목에서 지정된 바인딩과 서비스 동작을 사용하여 서비스를 초기화 하게 됩니다.

---