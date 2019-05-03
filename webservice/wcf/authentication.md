# WCF 서비스 인증

WCF 서비스의 인증은 HTTP와 WS-Security 등 W3C의 표준적인 방법을 통해 인증을 수행할 수 있습니다. 하지만 Fox Web Service는 서비스가 `FoxUserInfoContext` 객체를 기반으로 인증된 사용자만이 서비스를 호출할 수 있도록 제한이 가능합니다. 비록 비 표준적인 방법이지만 간단하며 사용자 아이디 뿐만 아니라 사용자에 대한 다양한 정보를 포함할 수 있기 때문에 효과적입니다.

WCF 서비스의 인증을 활성화하기 위해 Fox Web Service는 `FoxAuthenticationBehavior` 클래스나 `FoxAuthenticationAttribute` 클래스를 사용하여 인증을 활성화할 수 있습니다. Fox Web Service 인증이 활성화되면 서비스는 WCF 메시지에 `FoxUserInfoContext` 정보가 포함되어 있지 않다면 인증되지 않은 사용자로 간주하여 인증 오류를 유발합니다. 인증이 활성화된 서비스를 호출하기 위해서 클라이언트는 `FoxUserInfoContext` 객체를 통해 인증된 사용자 정보를 설정하고 `FoxClientFactory` 클래스를 통해 서비스를 호출해야 합니다.

## 서버측 인증 설정

기본적으로 WCF 서비스는 인증을 수행하지 않습니다. 따라서 인증되지 않은 클라이언트가 서비스를 호출할 수 있습니다. Fox Web Service 기반으로 인증된 클라이언트만이 서비스를 호출하도록 구성하기 위해서는 `FoxAuthenticationBehavior` 클래스나 `FoxAuthenticationAttribute` 클래스를 사용할 수 있습니다.

두 클래스는 모두 WCF의 Message Inspector 기능을 사용하여 서비스 Request 메시지 내에 `FoxUserInfoContext` 객체 정보가 포함되어 있는가를 확인합니다. `FoxUserInfoContext` 객체 정보가 포함되지 않은 경우 인증 오류(`FoxAuthenticationException`)를 발생하고 서비스가 호출되지 않습니다. `FoxUserInfoContext` 정보가 포함된 경우 이 정보로부터 `FoxUserInfoContext` 객체를 구성(deserialize)하여 서비스 코드 내에서 참조할 수 있도록 `FoxUserInfoContext.Current` 속성에 설정합니다. `FoxUserInfoContext` 클래스에 대한 상세한 내용은 [Fox Core 문서](core/README.md)를 참조하십시오.

### FoxAuthenticationBehavior 구성

`FoxAuthenticationBehavior` 클래스는 WCF의 동작(behavior) 기능을 활용한 커스텀 Behavior 클래스입니다. 커스텀 Behavior를 설정하기 위해서는 `web.config` 설정에 커스텀 Behavior를 등록해야 합니다. 다음은 `web.config` 파일에 `FoxAuthenticationBehavior` 클래스를 커스텀 Behavior로 등록하는 구성 설정의 예를 보여줍니다.

> WCF Behavior는 구성 설정뿐만 아니라 코드에 의해 추가될 수도 있습니다. 이 방법은 WCF 기본 코딩 방법에 해당하며 이 문서의 범위에서 벗어나므로 MSDN 혹은 WCF 관련 자료를 참고하십시오.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.serviceModel>
    <extensions>
      <behaviorExtensions>
        <add name="foxAuthentication" type="TheOne.ServiceModel.FoxAuthenticationBehavior, TheOne.ServiceModel.4.5, Version=4.5.0.0, Culture=neutral, PublicKeyToken=6895727a3cc10e00"/>
      </behaviorExtensions>
    </extensions>
  </system.serviceModel>
</configuration>
```

`<behaviorExtensions>` 요소에 `<add>` 요소에 추가하고 `type` 속성에 어셈블리 이름 및 네임스페이스를 포함하는 `FoxAuthenticationBehavior` 클래스의 전체 이름을 명시해야 합니다. 그리고 `name` 속성은 임의의 이름을 사용할 수 있습니다만 인증을 위한 확장임을 명시하도록 이름을 주는 것이 좋습니다.

커스텀 Behavior를 등록했다면 이제 서비스 동작, 즉 `<serviceBehaviors>`의 `<behavior>` 요소에 이 `FoxAuthenticationBehavior`를 XML 요소로 추가할 수 있습니다. 다음은 앞서 등록된 `FoxAuthenticationBehavior`를 사용한 서비스 동작 설정을 보여주고 있습니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.serviceModel>
    <behaviors>
      <serviceBehaviors>
        <behavior name="commonBehavior">
          <foxAuthentication allowAnonymous="false" />
        </behavior>
      </serviceBehaviors>
    </behaviors>
  </system.serviceModel>
</configuration>
```

위 구성 설정에서 `<foxAuthentication>` 요소의 이름은 `<behaviorExtensions>` 요소에 추가된 `<add>` 요소의 `name` 속성 값이 사용되었음에 주의하십시오. `name` 속성 값이 변경되면 `<foxAuthentication>` 요소의 이름도 변경되어야 합니다.

`<foxAuthentication>` 요소는 `allowAnonymous` 속성을 통해 인증되지 않은 사용자의 허용 여부를 설정할 수 있으며 기본값은 `false`입니다. 비록 기본값이 `false` 이더라도 명시적으로 `allowAnonymous` 속성의 값을 표시하는 것이 좋습니다.

이렇게 `FoxAuthenticationBehavior`를 서비스 동작으로 추가하면, 이 서비스 동작(위 예제에서는 `commonBehavior`)을 사용하는 WCF 서비스는 Fox Web Service 기반 인증이 사용되며 인증되지 않은 클라이언트의 호출은 인증 오류가 발생하게 됩니다. 서비스 동작을 사용하는 방법은 [바인딩 맵 선택](bindingmap.md#바인딩-맵-선택) 항목이나 WCF 관련 자료를 참고하십시오.

### FoxAuthenticationAttribute 구성

`FoxAuthenticationBehavior` 클래스를 등록하고 서비스 동작(behavior)을 추가하는 방식은 `web.config` 구성 설정만을 사용하여 서비스의 인증 여부를 설정합니다. 한편 `FoxAuthenticationAttribute` 클래스는 서비스의 소스 코드 상에서 서비스의 인증 여부를 설정합니다. `FoxAuthenticationAttribute` 클래스는 서비스 타입(서비스 인터페이스를 구현하는 클래스)에 추가할 수 있습니다. 다음은 `FoxAuthenticationAttribute` 클래스를 사용하여 서비스의 인증을 설정하는 예제를 보여줍니다.

> `FoxAuthenticationAttribute` 클래스는 `IServiceBehavior` 인터페이스를 구현하는 WCF 서비스 동작(behavior)의 한 종류입니다.

```csharp
// FoxAuthenticationAttribute를 이용한 인증 설정
[FoxAuthentication(AllowAnonymous = false)]
public class WcfService4 : IWcfService4
{
    ...... 생략 ......
}
```

`FoxAuthenticationAttribute` 클래스는 `FoxAuthenticationBehavior` 클래스와 동일하게 `AllowAnonymous` 속성을 명시하여 인증되지 않은 클라이언트 허용 여부를 설정할 수 있습니다. `AllowAnonymous` 속성의 기본값은 `false` 입니다만 위 코드와 같이 명시적으로 그 값을 지정해 주는 것이 좋습니다.

`FoxAuthenticationBehavior` 클래스와 `FoxAuthenticationAttribute` 클래스는 동시에 사용이 가능합니다. `Web.config`에서 `FoxAuthenticationBehavior`가 구성되었고 서비스 코드 상에서 `FoxAuthenticationAttribute` 클래스가 모두 명시된 경우, 코드 상에 명시된 `FoxAuthenticationAttribute` 설정이 우선됩니다. 예를 들어, `web.config` 에서는 인증된 클라이언트만 서비스 호출을 허용(`allowAnonymous = false`)하고 `FoxAuthenticationAttribute`에서는 인증되지 않은 클라이언트 서비스 호출도 허용(`allowAnonymous = true`)하는 경우 인증되지 않은 클라이언트 호출이 허용되게 됩니다.

> 인증을 설정하는 방법이 두 가지 존재하지만 권장되는 방법은 `FoxAuthenticationBehavior` 클래스를 이용하여 `web.config`의 구성 설정을 사용하는 것입니다. 인증이 필요한 서비스를 위한 서비스 동작(`serviceBehavior`)과 인증이 필요 없는 서비스 동작을 모두 구성하고 서비스에 따라 바인딩 맵 선택을 통해 인증 여부를 선택하도록 하면 됩니다. 바인딩 맵을 통해 서비스 동작을 선택하는 방법은 [바인딩 맵 선택](bindingmap.md#바인딩-맵-선택) 항목을 참고하십시오.

`FoxAuthenticationAttribute` 클래스는 인증을 적용할 수 없는 서비스에 대해 코드 상에서 인증 사용하지 않음을 강제할 때 유용합니다. 예를 들어, 로그인 과정을 거치기 전에 호출되는 서비스(사용자 아이디/암호 확인 등)는 로그인이 아직 안된 상황이기 때문에 인증을 사용할 수 없습니다. 따라서 이러한 서비스에 `FoxAuthenticationAttribute` 클래스를 적용하고 `AllowAnonymous` 속성을 `true`로 지정하면 됩니다. `FoxAuthenticationAttribute` 클래스 설정은 `web.config`의 구성 설정보다 우선되므로 항상 익명 클라이언트의 호출을 허용하게 됩니다. 다음 코드는 이와 같은 예를 보여줍니다.

```csharp
[FoxAuthentication(AllowAnonymous = true)]
public class Login : ILogin
{
    public bool LoginUser(string userId, string password)
    {
        ... 생략 ...
    }

    ... 생략 ...
}
```

## 클라이언트 측 인증 설정

WCF 서비스가 인증을 요구하면 클라이언트는 인증 정보를 설정하고 설정된 인증 정보를 WCF 서비스 호출에 포함시켜야 합니다. 인증 정보 설정은 `FoxUserInfoContext` 객체를 생성하고 이 객체의 `SetCallContext` 메서드를 호출해야 합니다. 다음은 이와 같은 과정을 보여주는 간단한 예제 코드입니다.

```csharp
var ctx = new FoxUserInfoContext("TestUser");
ctx["DeptId"] = "TestDept";
ctx.SetCallContext();
```

위 코드는 사용자 아이디(`TestUser`)와 사용자 정보(`DeptId`)를 하드 코딩하였지만 실제 어플리케이션에서는 로그인 과정을 통해 사용자의 아이디를 입력 받고 사용자가 인증된 경우, 사용자 정보를 DB에서 읽어 `FoxUserInfoContext` 객체에 설정하게 됩니다.

> `SetCallContext` 메서드 호출은 매우 중요합니다. 이 메서드 호출은 `FoxUserInfoContext` 객체를 스레드 로컬 저장소(TLS; Thread Local Storage)에 기록합니다.

클라이언트가 [FoxClientFactory 클래스](clientfactory.md)를 통해 서비스를 호출하면 스레드 로컬 저장소에 기록된 `FoxUserInfoContext` 객체를 직렬화(serialize) 하여 서비스 호출 메시지에 데이터로 포함시키게 됩니다. 인증이 활성화된 서비스는 호출 메시지에 포함된 `FoxUserInfoContext` 객체 데이터의 포함 여부로 인증 여부를 판단하게 됩니다.

> `FoxClientFactory` 클래스를 사용하지 않는 경우 클라이언트 측 `app.config`에 `FoxAuthenticationBehavor`를 구성하고 이를 종점 동작(endpoint behavior)에 포함시키면 됩니다.

## 인증된 사용자 정보 획득

서비스 측에서 인증된 사용자 정보를 획득하기 위해서는 `FoxUserInfoContext.Current` 속성에 접근하면 됩니다. `FoxAuthenticationBehavior` 클래스 혹은 `FoxAuthenticationAttribute` 클래스를 통해 인증 기능이 작동하면 클라이언트가 호출할 때 포함시킨 인증 정보를 역직렬화(deserialize)하여 `FoxUserInfoContext` 객체를 구성하고 이 객체를 스레드 로컬 저장소에 기록해 둡니다. 이렇게 스레드 로컬 저장소에 기록된 `FoxUserInfoContext` 객체에 접근하는 속성이 `FoxUserInfoContext.Current` 속성인 것입니다. 다음 코드는 서비스를 호출한 클라이언트의 정보를 획득하는 예제를 보여주고 있습니다.

```csharp
[FoxAuthentication(AllowAnonymous = false)]
public class WcfService4 : IWcfService4
{
    public string EchoUserInfo()
    {
        // 호출한 사용자를 나타내는 UserInfoContext 객체 획득
        var ctx = FoxUserInfoContext.Current;
        if (ctx == null)
        {
            throw new UnauthorizedAccessException("인증되지 않은 사용자");
        }
        return $"UserId:{ctx.UserId},  DeptId={ctx["DeptId"]}";
    }
}
```

이렇게 획득한 사용자 정보는 서비스를 호출한 사용자를 나타내므로 로깅, DB 변경자 기록 등 다양한 목적으로 사용할 수 있습니다. `FoxUserInfoContext` 객체에는 단순한 사용자 아이디 뿐만 아니라 클라이언트가 설정한 추가적인 정보도 포함되어 있습니다. 위 예제 코드에서는 부서 코드(`DeptId`)를 추출하고 있습니다. 이러한 추가정보는 물론 클라이언트가 설정해 놓아야 하며 이는 어플리케이션마다 달라질 수 있습니다.

---