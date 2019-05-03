# 클라이언트 팩터리(FoxClientFactory)

Fox Web Service는 Visual Studio의 서비스 참조 기능을 사용하지 않고 다수의 서비스를 사용할 수 있도록 다양한 기능을 지원합니다. `FoxClientFactory` 클래스는 Fox Web Service 기반의 서비스를 호출하기 위해 필요한 작업들을 단순화시켜주는 다양한 기능을 포함합니다. 이 클래스의 대표적인 기능은 WCF 클라이언트(WCF `ChannelFactory` 및 `ClientChannel`) 생성과 Fox Biz/Data Service 클라이언트 생성입니다. 이 섹션에서는 Fox Biz/Data Service 관련 기능을 제외한 나머지 기능을 살펴볼 것이며 Fox Biz/Data Service 관련 기능들은 추후 [Fox Biz Service](/webservice/bizservice/README.md), [Fox Data Service](/webservice/dataservice/README.md) 기능을 설명할 때 다시 언급하도록 하겠습니다.

## WCF 클라이언트 구성

Visual Studio의 서비스 참조 기능을 사용하지 않고 서비스를 호출하기 위해 WCF 클라이언트는 `ChannelFacotry` 객체를 생성하고 생성한 `ChannelFactory` 객체로부터 `ClientChannel` 객체를 생성하여 서비스를 호출하게 됩니다. 만약 서비스가 인증 기능([WCF 서비스: 클라이언트 측 인증](authentication.md#클라이언트-측-인증-설정) 참조)을 사용하는 경우 인증을 위한 WCF 동작(behavior) 설정도 필요합니다. 이러한 과정은 많은 코드를 요구하며 서비스 호출할 때마다 반복적으로 수행되어야 합니다.

> 서비스 참조와 Fox Web Service를 모두 사용하지 않고 서비스를 호출하는 코드 예제는 이 문서의 범위를 벗어나므로 WCF 관련 문서나 도서를 참고하십시오.

### 기본 호출 방식

`FoxClientFactory` 클래스는 WCF 서비스 호출에 필요한 과정을 미리 구현하여 제공하여 `CreateChannel<T>` 메서드를 호출하면 필요한 여러 과정을 수행한 후에 즉시 서비스 호출이 가능한 서비스 인터페이스를 반환해 줍니다.

다음 코드는 `FoxClientFactory.CreateChannel<T>` 메서드 호출을 통해 WCF 서비스를 호출하는 가장 간단한 예를 보여줍니다.

```csharp
// FoxClientFactory를 사용하여 WCF 서비스 호출
using (var svc = FoxClientFactory.CreateChannel<IWcfService>("WcfService.svc"))
{
    var ds = svc.GetAllProducts();
    grdProducts.DataSource = ds.Tables[0];
}
```

위 코드는 `CreateChannel<T>`의 오버로드된 메서드들 중 가장 간단한 형태입니다. [디폴트 주소 맵](addressmap.md#주소-맵-구성)과 [디폴트 바인딩 맵](bindingmap.md#바인딩-맵-구성)을 사용하고, 서비스에 대한 상대 주소만을 사용하여 서비스를 호출하게 됩니다.

다음 코드는 명시적으로 주소 맵의 이름과 바인딩 맵의 이름을 사용하고 상대 주소를 사용하여 서비스를 호출하는 예제 코드 입니다.

```csharp
// 다른 구성 설정으로 WCF 서비스 호출
using (var svc = FoxClientFactory.CreateChannel<IWcfService>("baseAddr", "WcfService1.svc", "netHttp"))
{
    var ds = svc.GetAllProducts();
    grdProducts.DataSource = ds.Tables[0];
}
```

### 고급 호출 방식

`FoxClientFactory.CreateChannel<T>` 메서드는 주소 맵이나 바인딩 맵을 사용하지 않고도 서비스를 호출할 수 있도록, 추가적인 메서드 오버로드를 제공합니다. 이들 메서드들은 구성 설정 파일을 사용하지 않고 코드에 의해 동적으로 서비스 호출을 구성하는 경우 필요합니다. 다음 코드는 전체 URL 주소와 바인딩 객체를 직접 사용하여 서비스를 호출하는 코드를 보여줍니다.

```csharp
// 주소 맵이나 바인딩 맵을 사용하지 않는 서비스 호출 코드
var url1 = "http://localhost:32900/wcfservice.svc";
var binding = new BasicHttpBinding();
using (var svc = FoxClientFactory.CreateChannel<IWcfService>(url1, binding))
{
    var ds = svc.GetAllProducts();
    grdProducts.DataSource = ds.Tables[0];
}
```

다음 코드는 바인딩 이름과 종점 동작(endpoint behavior) 이름을 사용하여 서비스 호출을 수행하는 예제 코드 입니다. 이 코드에서 사용된 바인딩 이름(`defaultBinding`)과, 종점 동작 이름은 app.config 파일에 구성되어 있어야 합니다.

```csharp
var url2 = "http://localhost:32900/login.svc";
var ep = new EndpointAddress(url2);
var bindingName = "defaultBinding";
using (var svc = FoxClientFactory.CreateChannel<ILogin>(ep, bindingName, null))
{
    if (svc.LoginUser("tester", "passport") == false)
    {
        ... 생략 ...
    }
}
```

## 클라이언트 측 인증

`FoxClientFactory` 클래스를 사용하여 서비스를 호출하는 경우, 클라이언트 측의 사용자 정보(`FoxUserInfoContext`)가 호출 메시지에 포함됩니다. 이를 통해 [사용자 인증을 요구하는 WCF 서비스](authentication.md)를 오류 없이 호출 할 수 있습니다. 상세한 내용은 [클라이언트 측 인증 설정](authentication.md#클라이언트-측-인증-설정) 항목을 참고 하십시오.

## 다른 기능

`FoxClientFactory` 클래스는 일반적인 WCF 서비스 뿐만 아니라 `Fox Biz Service` 및 `Fox Data Service`에 대한 클라이언트 팩터리 역할도 수행합니다. `CreateBizClient` 및 `CreateDataClient` 메서드를 사용하여 `Fox Biz Service` 클라이언트와 `Fox Data Service` 클라이언트 객체를 생성할 수 있습니다. 이 기능에 대해서는 [Fox Biz Service](/webservice/bizservice/README.md) 항목과 [Fox Data Service](/webservice/dataservice/README.md) 항목을 참고 하십시오.

---