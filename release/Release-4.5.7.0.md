# NeoDEEX 4.5.7 Change Log

NeoDEEX 4.5.7.0 버전에 대한 변경 사항들 중에서 주목해야 할 사항들을 나열한 것입니다. **이 문서에서 언급되지 않은 변경사항**이 있을 수도 있으며, 이 문서에서 언급된 변경 사항 역시 예고 없이 변경될 수 있습니다.

---

## Breaking Change

이번 릴리스에는 하위 호환성을 해치는 변경 사항이 없습니다.

## PostgreSQL 데이터 베이스 지원

Fox DB Access에 PostgreSQL 데이터 베이스 지원(`TheOne.Data.Npgsql.FoxNpgsqlDbAccess`)이 추가되었습니다.

* PostreSQL 프로바이더를 위해 `NeoDEEX.Data.Npgsql.dll` 어셈블리가 추가되었습니다. 베타 버전인 관계로 이 어셈블리는 설치 패키지에 포함되지 않으며 별도의 복사가 필요합니다.

  * :warning: NeoDEEX.Data.Npgsql.dll 어셈블리를 사용하기 위해서는 **.NET Framework 버전 4.7.2 이상의 버전을 사용**해야 합니다.

  * `NeoDEEX.Data.Npgsql.dll` 어셈블리는 `Npgsql` 버전 5.0.13을 사용하여 빌드되었습니다. 이 어셈블리를 사용하기 위해서는 `Npgsql` NuGet 패키지 5.0.13 버전 이상을 프로젝트에 추가해야 합니다.

    > :warning: `Npgsql` NuGet 패키지는 System.Text.Json 등 다양한 추가 패키지에 대해 의존성을 갖습니다.

    * `Npgsql` 버전 5.0.13 이상의 버전을 사용하는 경우 `Npgsql.dll` 어셈블리와 이 어셈블리가 의존하는 어셈블리에 대해 바인딩 리디렉션이 필요합니다. 다음은 `Npgsql` 6.0.4 버전을 사용했을 때 필요한 바인딩 리디렉션의 예 입니다.

    ```xml
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="Npgsql" publicKeyToken="5d8b90d52f46fda7" culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-6.0.4.0" newVersion="6.0.4.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="System.Runtime.CompilerServices.Unsafe" publicKeyToken="b03f5f7f11d50a3a" culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-6.0.0.0" newVersion="6.0.0.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="System.Buffers" publicKeyToken="cc7b13ffcd2ddd51" culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-4.0.3.0" newVersion="4.0.3.0" />
      </dependentAssembly>
    </assemblyBinding>
    ```

    > :information_source: 사용하는 `Npgsql` 버전에 따라서 바인딩 리디렉션의 항목 개수, 버전 등은 변경될 수 있습니다.

## 동적 Fox Query 베타

Fox Query의 SQL 문장을 동적으로 변경할 수 있는 동적 Fox Query 기능 베타 버전이 추가되었습니다.

:warning: 동적 쿼리 기능을 사용하려면 `Z.Expression.Eval.dll` 어셈블리를 필요로 합니다. 

  ```xml
    <statement id="SimpleMacroTest">
      <text>
        SELECT ProductId, ProductName, $$SelectMacro()$$ FROM UnitTest_Products $$WhereMacro()$$
      </text>
      <parameters>
        <parameter name="CategoryName" dbType="VarChar"/>
      </parameters>
      <macros>
        <macro name="SelectMacro">
          if (env.Args.CategoryName != null)
            return "UnitPrice";
          else
            return "CategoryName, UnitPrice";
        </macro>
        <macro name="WhereMacro">
          if (env.Args.CategoryName != null)
          {
            env.Args.CategoryName = env.Args.CategoryName.SurroundWith("%");
            return $"WHERE CategoryName LIKE #CategoryName#";
          }
          else
          {
            env.Params.Remove("CategoryName");
            return null;
          }
        </macro>
      </macros>
    </statement>
  ```

* 동적 Fox Query는 매크로 방식으로 작동하며 SQL 문장에 매크로를 삽입할 수 있습니다. 매크로는 `$$` 문자를 사용하여 표시합니다. 위 예에서 `<text>` 요소 내부의 `$$SelectMacro()$$` 와 `$$WhereMacro()$$` 가 매크로에 해당됩니다.

* 매크로 정의는 새로이 추가된 `<macros>` 요소를 통해 정의 할 수 있습니다. 각 매크로는 C# 코드를 통해 매크로가 어떻게 치환(replace) 될지 코드를 작성하고 문자열을 반환하면 됩니다.

* 매크로 C# 스크립트에서 `env` 객체는 Fox Query에서 정의한 매개변수와 Fox Query가 수행될 때 전달되는 매개변수 값에 접근할 수 있는 속성을 제공합니다.

  * `env.Params` 속성은 Fox Query에서 `<parameters>` 요소에 정의된 매개변수들 정보에 대한 컬렉션을 제공합니다.
  * `env.Args` 속성은 Fox Query 수행 시 전달된 매개변수 값들에 대한 컬렉션을 제공합니다.
  * `env.Log` 속성은 스크립트에서 로그를 기록하고자 할 때 사용할 수 있는 `IFoxLog` 인터페이스 입니다.

* 스트립트 로거 설정은 `<queryMapper>` 에 신규로 추가된 `<script>` 요소를 사용하여 로거 이름을 지정할 수 있습니다.

```xml
  <queryMapper name="SchemaTestMap">
    <settings>
      <typeConverter type="TheOne.Data.Extensions.FoxTypeConverter" />
      <script loggerName="ScriptLogger"/>
    </settings>
    <queryMaps>
      <files>
          <file path="./Query/SimpleTest.foxml" />
      </files>
      <directories>
        <directory path="./" includeSubdirectories="true" />          
      </directories>
    </queryMaps>
  </queryMapper>
```

## 버그 수정

이번 릴리즈에는 버그 수정 사항이 없습니다.

---
