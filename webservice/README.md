# Fox Web Services 개요

Fox Web Services는 다 계층(n-tier) 분산 어플리케이션에서 클라이언트와 서버 사이의 연결을 지원하는 NeoDEEX의 네트워킹 연결 기능입니다. Fox Web Services는 Fox Data Service를 통해 클라이언트가 DB 액세스를 손쉽게 수행할 수 있도록 해주며 Fox Biz Service를 통해 비즈니스 로직 호출에서 서비스 계층을 작성하지 않아도 되도록 해줍니다. 또한, 서비스 계층을 작성해야 할 때에도 복잡한 구성 설정 없이 WCF를 통해 서버에 접근할 수 있도록 해줍니다.

![Fox Web Service 개요](images/readme-1.png)  
[그림1. Fox Web Services 개요]

이 문서는 Fox Web Services에서 제공하는 WCF 지원 기능, Fox Biz Service, Fox Data Service에 대한 설명을 제공합니다.

개발자 가이드 목차

* [전통적인 WCF 서비스 기반 어플리케이션의 문제점](wcf/problems.md)

* [Fox Web Services를 사용하는 WCF 서비스 구성 예제](wcf/step-by-step.md)

  * [How-to: WCF 서비스 구성](wcf/howto-service.md)

  * [How-to: WCF 클라이언트 구성](wcf/howto-service.md)

* [WCF 지원 기능들](wcf/features.md)

  * 서비스 호스트 팩터리

  * 바인딩 맵

  * 주소 맵

  * 서비스 인증

  * FoxClientFactory

  * 메시지 압축

* [Fox Biz Service](webservice/bizservice/README.md)

  * [서버 구성](webservice/bizservice/serverconfig.md)

    * [How To : WCF 서비스 구성](webservice/bizservice/howto-wcf.md)

    * [How To : REST API 서비스 구성](/webservice/bizservice/howto-rest.md)

* [Fox Data Service](webservice/dataservice/README.md)

  * [서버 구성](webservice/dataservice/serverconfig.md)

    * [How To : WCF 서비스 구성](webservice/dataservice/howto-wcf.md)

    * [How To : REST API 서비스 구성](/webservice/dataservice/howto-rest.md)

---