# Fox Web Services의 WCF 지원 기능

NeoDEEX에서 클라이언트가 서버를 호출하여 비즈니스 로직이나 데이터베이스에 액세스하고자 할 때 표준적으로 제시하는 서버 통신 메커니즘은 WCF(Windows Communication Foundation) 입니다. 하지만 대형 어플리케이션에서 WCF를 사용할 때 다양한 문제점들이 발생할 수 있습니다. Fox Web Service는 이러한 문제점들을 해결하는 다양한 기능들을 제공합니다. 구체적인 예제에서 이 이들 기능을 간략히 살펴 본 후, 각 기능들의 상세한 내용을 설명하겠습니다.

목차

* [전통적인 WCF 서비스 기반 어플리케이션의 문제점](problems.md)

* [Fox Web Services를 사용하는 WCF 서비스 구성 예제](step-by-step.md)

  * [How-to: WCF 서비스 구성](howto-service.md)

  * [How-to: WCF 클라이언트 구성](howto-client.md)

* [WCF 지원 기능들](features.md)

  * 서비스 호스트 팩터리

  * 바인딩 맵

  * 주소 맵

  * 서비스 인증

  * FoxClientFactory

  * 메시지 압축

---