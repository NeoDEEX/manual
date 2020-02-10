# NeoDEEX 4.5.5 Change Log

NeoDEEX 4.5.5.x 버전에 대한 변경 사항들 중에서 주목해야 할 사항들을 나열한 것입니다. **이 문서에서 언급되지 않은 변경사항**이 있을 수도 있으며, 이 문서에서 언급된 변경 사항 역시 예고 없이 변경될 수 있습니다.

---

## Breaking Change

* Fox Biz/Data Service REST API 클라이언트의 기본 타임아웃 값이 100초에서 300초(5분)로 변경되었습니다.

## Fox Biz/Data Service REST API 클라이언트의 타임아웃 설정 기능 추가

Fox Biz/Data Service REST API 클라이언트의 타임아웃 설정 기능이 추가 되었습니다.

* `FoxRestBizClient` 및 `FoxRestDataClient` 클래스에 `Timeout` 속성이 추가 되었습니다.

  * `Timeout` 속성은 클라이언트의 타임아웃 값으로 초 단위로 설정이 가능합니다. 이 속성의 기본값은 300초 입니다.

    ```cs
    var client = new FoxRestBizClient(...);
    client.Timeout = 600;
    ```

* REST API 클라이언트의 전역 기본 타임아웃 값을 설정할 수 있는 구성 설정이 추가되었습니다.

  * `<service>` 항목의 `<restAPI>` 요소에 `clientTimeout` 속성이 추가되었습니다. 이 속성이 지정되지 않으면 300초의 기본값이 사용되며 여기에 지정된 값은 `FoxRestBizClient` 및 `FoxRestDataClient` 클래스의 `Timeout` 속성의 기본값으로 사용됩니다.

    ```xml
    <service>
      <restAPI clientTimeout="600"/>
    </service>
    ```

> 주) Fox Biz/Data Service REST API 서버측 타임 아웃은 서비스가 호스티되는 환경에 맞추어 타임 아웃을 설정해야 합니다. 대표적으로 IIS, IIS Express가 사용되는 경우, `web.config` 파일에 <system.web/httpRuntime> 요소에 `executionTimeout` 속성을 사용합니다(기본값 90초).

```xml
<system.web>
  <httpRuntime executionTimeout=600>
</system.web>
```

## 버그 수정

이 버전에 포함된 버그 수정 사항이 없습니다.

---
