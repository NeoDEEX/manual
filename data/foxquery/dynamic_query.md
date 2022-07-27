# 동적 쿼리

:warning: 동적 쿼리를 사용할 때에는 대단한 주의가 필요합니다. 동적 쿼리는 SQL 문장을 동적으로 변환할 수 있기 때문에 클라이언트로 부터 전달된 매개변수로 부터 SQL Injection 공격을 받을 수 있습니다. 따라서 동적 쿼리가 SQL Injection 문제를 유발하지 않도록 쿼리를 구사해야 합니다.

Fox Query 에서 동적 쿼리를 사용하는 방법에 대해 설명합니다. 동적 쿼리라 함은 Fox Query 에서 수행하는 SQL 문장이 상황에 따라 변경될 수 있음을 말합니다. 예를 들어 클라이언트가 어떤 컬럼에 대한 조건을 제시한 경우에는 WHERE 절에 조건을 사용하고 그렇지 않은 경우 모든 데이터를 조회해야 하는 상황을 생각해 볼 수 있습니다. Fox Query 가 동적 쿼리를 제공하지 않는다면 2개의 쿼리를 작성하고 매개변수 제시 여부에 따라 2개 중 하나를 선택하도록 로직을 작성해야 합니다. 다음 예제는 동적 쿼리를 사용하지 않을 때의 예제 코드와 Fox Query 의 예를 보여줍니다.

```cs
public DataSet GetData(int? categoryId)
{
    if (categoryId == null)
    {
        return this.DbAccess.ExecuteQueryDataSet("LegacyQuery.GetAll");
    }
    else
    {
        var parameters = new { CategoryId = categoryId };
        return this.DbAccess.ExecuteQueryDataSet("LegacyQuery.GetByCategory", parameters);
    }
}
```

```xml
<statement id="GetAll">
  <text>
    SELECT ProductID, ProductName, CategoryID FROM Products
  </text>
</statement>
<statement id="GetByCategory">
  <text>
    SELECT ProductID, ProductName, CategoryID FROM Products WHERE CategoryID = #CategoryId#
  </text>
  <parameters>
    <parameter name="CategoryId"/>
  </parameters>
</statement>
```

> :information_source: 위 예제와 같은 SQL 문장은 동적 쿼리를 사용하지 않고
> IS NULL 과 OR 연산자를 사용하여 단일 SQL 문장으로 구사가 가능합니다만
> 동적 쿼리에 대한 간단한 예제로 이해해 주십시요.

Fox Query 와 같은 쿼리 매퍼들은 동적 쿼리를 다양한 방법으로 지원합니다. 예를 들어 myBatis 는 SQL 문장 내에 XML 태그를 사용하여 조건절, 반복문을 사용하여 동적인 SQL 문장 생성을 지원합니다.

:memo: 특히 Fox Data Service 를 사용하는 상황에서 동적 쿼리가 제공되지 않는다면 반드시 위 코드와 같은 비즈니스 로직 컴포넌트를 서버 측에 작성해야만 합니다. 반면 **동적 쿼리를 사용하면 간단한 SQL 구성 로직은 Foxml 내에 구현이 가능하므로 Fox Data Service 만을 사용하여 원하는 SQL 문장을 구사할 수 있습니다**.

## Fox Query 매크로

Fox Query 는 동적 쿼리 지원을 위해 매크로 기능을 제공합니다. Fox Qury 매크로는 C# 코드를 사용하여 매크로 함수를 정의하고 SQL 문장에서 매크로 함수가
사용되면 매크로 함수를 구동하고 그 결과값(문자열)으로 SQL 문장을 구성하는 방식입니다.

Fox Query 의 동적 쿼리는 다른 쿼리 매퍼들과 달리 C# 스크립트 코드를 사용하기 때문에 XML 태그의 제약 없이 C# 과 닷넷의 풍부한 프로그래밍 기능을 모두 사용할 수 있어서 보다 자유로운 SQL 문장 재구성이 가능합니다.

### 매크로 정의

Fox Query 매크로를 정의하기 위해 `<macros>` 태그와 `<macro>` 태그가 Foxml 에 새롭게 추가되었습니다. 이 두 태그는 `<parameters>` `<parameter>` 태그와 유사하게 `<statement>` 태그 내에 사용할 수 있습니다.

다음은 매크로를 정의하는 foxml 조각을 보여주고 있습니다. 다음 매크로는 `MYMACRO` 매크로를 정의하고 있습니다. 이 매크로는 `CategoryId` 인자(argument) 값이 `null` 인가를 확인하여 `null` 이 아닌 경우, 매개변수 `CategoryId` 를 추가하고 `WHERE` 절을 구성하여 반환합니다. 사용된 C# 스크립트 코드는 런타임에 컴파일되고 수행됩니다. 컴파일된 코드는 내부적으로 캐시되므로 반복적으로 컴파일되지 않습니다.

```xml
<macros>
  <macro name="MYMACRO">
    if (env.Args.CategoryId == null) {
      return null;
    }
    else {
      var p = env.Params.Add("CategoryId");
      p.DbTypeName = "Int";
      return "WHERE CategoryId = #CategoryId#";
    }
  </macro>
</macros>
```

매크로는 SQL 문장을 변경하는 것외에도 Fox Query 의 매개변수를 추가/수정/변경 할 수도 있습니다. Fox Query 의 매개변수는 `<parameters>` 태그에 의해 정의되는 쿼리의 매개변수 목록입니다. 매개변수를 수정하기 위해서는 `env.Params` 컬렉션 속성을 사용하여 추가/수정/삭제가 가능합니다. 상세한 내용은 추후 이 문서에서 상세히 언급될 것입니다.

### 매크로 호출식

작성된 매크로를 사용하기 위해서는 SQL 문장에서 매크로 이름과 괄호, `$$` 문자를 사용하여 호출식을 사용하면 됩니다. 위에서 작성한 `MYMACRO` 의 경우 표현식은 `$$MYMACRO()$$` 가 됩니다.

```xml
<text>
  SELECT ProductID, ProductName, CategoryID FROM Products
  $$MYMACRO()$$
</text>
```

Fox Query 엔진은 매크로 호출식을 발견하면 정의된 매크로들 중에서 매크로를 찾아 구동하고 매크로가 반환한 문자열을 호출식에 삽입합니다. 이 예제에서는 CategoryId 매개변수 인자값이 `null` 인 경우 SQL 문장은 변경되지 않고 `null` 이 아닌 경우 `WHERE` 절이 추가되게 됩니다.

### 매크로 예제

다음은 Fox Query 매크로를 사용하여 동적 쿼리를 구성하는 foxml 예제를 보여줍니다. `<macros>` 태그와 `<macro>` 태그를 사용하여 `WHERE` 매크로를 정의하고 SQL 문장 내에서 `$$WHERE()$$` 호출식을 사용하였습니다.

```xml
<statement id="GetData">
  <text>
    SELECT ProductID, ProductName, CategoryID FROM Products
    $$WHERE()$$
  </text>
  <macros>
    <macro name="WHERE">
      if (env.Args.CategoryId == null) {
        return null;
      }
      else {
        env.Params.Add("CatetoryId", "Int");
        return "WHERE CategoryId = #CategoryId#";
      }
    </macro>
  </macros>
</statement>
```

`WHERE` 매크로는 CategoryId 인자값을 검사하여 `null` 이 아닌 경우에만 `WHERE` 절을 SQL 문장에 삽입합니다. Fox Query 의 매크로 기능을 사용함으로써 Fox Query를 구동하는 C# 코드는 이제 다음과 같이 작성이 가능합니다.

```cs
public DataSet GetData(int? categoryId)
{
    var parameters = new { CategoryId = categoryId };
    return this.DbAccess.ExecuteQueryDataSet("Query.GetData", parameters);
}

```

## 매크로 수행 환경

매크로 코드를 지원하기 위해 Fox Qurery 는 여러가지 기능을 지원합니다. 이 기능들은 Fox Query 에서 정의된 매개변수 정보, 쿼리 수행을 위해 전달된 매개변수 인자값들, 매크로 문제 해결 및 디버깅을 위한 로깅 입니다.

### 수행 환경 객체

매크로 코드를 지원하기 위한 거의 모든 기능들은 `TheOne.Data.Extensions.Script` 네임스페이스의 `FoxQueryScriptEnvironment` 클래스를 통해 제공됩니다. 이 클래스는 다음과 같은 속성/메서드들을 제공합니다.

* `Params` 속성

  Fox Query에 정의된 매개변수에 대한 컬렉션을 제공합니다.

* `Args` 속성

  Fox Query 수행을 위해 제공된 매개변수 인자값들에 대한 컬렉션을 제공합니다.

* `Log` 속성

  로그 기록을 위한 `IFoxLog` 인터페이스 입니다.

* `WriteLog` 메서드

  `Verbose` 로그 수준으로 로그를 기록합니다.

Fox Query 엔진은 이 클래스의 인스턴스를 자동으로 생성하며 매크로 코드는 `env` 변수를 통해 접근이 가능합니다.

### 매개변수 컬렉션

`env.Params` 속성을 통해 접근이 가능한 Fox Query에 정의된 매개변수들에 대한 컬렉션입니다. 즉, Foxml 의 `<parameters>` 태그에 정의된 Fox Query 매개변수 정의에 대해 접근이 가능합니다. `Params` 속성은 매개변수들을 추가/삭제/수정할 수 있도록 다양한 속성과 메서드들 제공합니다.

* `Add`/`Remove` 메서드

  Fox Query 매개변수 컬렉션에 새로운 매개변수를 추가하거나 기존 매개변수를 제거할 수 있습니다. `Add` 메서드는 추가된 매개변수를 나타내는 `FoxParameter` 객체를 반환합니다. `FoxParameter` 클래스의 `Name`, `Property`, `DbTypeName` 등의 속성은 `<parameter>`태그의 `name`, `property`, `dbType` 등의 속성을 나타냅니다.

  ```cs
  var p1 = env.Params.Add("p1");
  p1.DbTypeName = "Varchar2";
  p1.Size = 50;
  var p2 = env.Params.Add("p2", "Varchar2");
  p2.Size = 50;
  env.Params.Add("p3", "Varchar2", 50);
  env.Params.Remove("p2");
  ```

* 인덱서 속성

  `Params` 속성에 인덱서를 사용하여 정의된 매개변수 속성을 참조하거나 변경할 수 있습니다.

  ```cs
  env.Params["p1"].Direction = ParameterDirection.InputOutput;
  ```

* 매개변수 이름 속성

  `Params` 속성은 C# 의 동적 객체(dynamic object) 기능을 지원하므로 인덱서를 사용하지 않고 매개변수의 이름을 속성 처럼 사용하여 접근이 가능합니다.

  ```cs
  env.Params.p1.Direction = ParameterDirection.IntputOutput;
  ```

* 매개변수 확인

  `ContainsKey` 메서드를 통해 특정 이름의 매개변수가 존재하는지 파악이 가능합니다.

  ```cs
  if (env.Params.ContainsKey("p2") == false)
  {
    env.Params.Add("p2", "clob", 128);
  }
  ```

* 열거 기능

  `Params` 속성은 매개변수에 대한 열거 기능을 제공합니다.

  ```cs
  foreach(FoxParameter p in env.Params)
  {
    p.DbTypeName = "Varchar2";
  }
  ```

### 인자값 컬렉션

`env.Args` 속성은 Fox Query 수행을 위해 전달된 매개변수 인자값들에 대한 컬렉션을 제공합니다. 즉, Fox Query 매크로는 `ExecuteQueryXXX` 메서드에 전달된 매개변수 인자값들을 `env.Args` 속성을 통해 액세스가 가능합니다. `env.Args` 속성을 통해 인자값에 따라 SQL 문장을 동적으로 구성할 수 있습니다. `Args` 속성은 다음과 같은 속성과 메서드를 제공합니다.

* 인덱서 속성

  `Args` 속성에 인덱서를 사용하여 매개변수 인자값에 접근이 가능합니다.

  ```cs
  env.Args["p1"] = env.Args["p1"] + "_CODE";
  ```

* 매개변수 이름 속성

  `Args` 속성은 C# 의 동적 객체(dynamic object) 기능을 지원하므로 인덱서를 사용하지 않고 매개변수의 이름을 속성 처럼 사용하여 접근이 가능합니다.

  ```cs
  env.Args.p1 = env.Args.p1 + "_CODE";
  ```

* `Add`/`Remove` 메서드

  매개변수 인자값을 추가하거나 제거합니다.

  ```cs
  env.Args.Add("p4", "Value");
  env.Args.Remove("p1");
  ```

  :warning: Fox Query에 전달된 매개변수 인자값이 `IDictionary` 인 경우에만 호출이 가능합니다. Fox Data Service는 `FoxServiceParameterCollection` 클래스가 `IDictionary` 인터페이스를 지원하므로 항상 `Add`/`Remove` 메서드를 호출할 수 있습니다.

* 열거 기능

  `Args` 속성은 매개변수의 키에 대한 열거 기능을 제공합니다. 매개변수 인자값으로 `IDictionary`, `DataRow`, 엔티티가 사용될 수 있으므로 열거 기능으로 나열되는 키는 `IDictionary` 에 대한 키, `DataRow` 의 컬럼 이름 혹은 엔티티의 속성 이름이 나열됩니다.

  ```cs
  foreach(string key in env.Args)
  {
    if (env.Args[key] == null)
    {
      env.Args[key] = DBNull.Value;
    }
  }
  ```

### 로깅

Fox Query 매크로는 3rd Party C# 스크립트 엔진(Z.Expressions.Eval)을 사용하기 때문에 디버깅이 어렵습니다. Fox Query 는 매크로 디버깅 및 문제 해결에 도움이 되도록 로깅을 지원합니다.

* 로깅 설정

  매크로에서 로깅을 사용하려면 로그가 기록될 로거 이름을 설정해 주어야 합니다. 쿼리 매퍼 구성 설정에서 `<settings>` 요소의 하위에 새롭게 추가된 `<script>` 요소의 `loggerName` 속성을 사용하면 됩니다.

  ```xml
  <queryMapper name="Default">
    <settings>
      <script loggerName="ScriptLogger"/>
    </settings>
    <queryMaps>
      <directories>
        <directory path="./foxml"/>
      </directories>
    </queryMaps>
  </queryMapper>
  ```

* `Log` 속성

  `env.Log` 속성은 `<script>` 요소의 `loggerName` 속성에 의해 지정된 로거를 나타내는 IFoxLog 인터페이스 입니다. 로거가 지정되지 않은 경우, Log 속성은 `FoxDummyLogger` 에 대한 인터페이스이며 로그를 기록하지 않습니다.

  ```cs
  env.Log.Information("Script log");
  ```

* `WriteLog` 메서드

  `WriteLog` 메서드는 보다 쉽게 로그를 남길 수 있도록 해주는 헬퍼 메서드로서 Fox Query 아이디와 매크로 이름을 자동으로 포함하며 `Verbose` 로그 수준으로 로그를 기록해 줍니다. 일반적으로 개발자는 이 메서드를 사용하여 로그를 기록하는 것이 권장됩니다.

  ```cs
  env.WriteLog($"p1 = {env.Args.p1}");
  ```

  ```log
  V 2022-01-01 01:01:01.50000 [ScriptLogger] Query.GetData.WHERE()> p1 = 2
  ```

### SQL 헬퍼 메서드

Fox Query 는 매크로에서 SQL 문장을 동적으로 구성하는데 도움이 되는 몇 가지 헬퍼 메서드들을 제공합니다. 이들 헬퍼 메서드는 FoxQueryScriptSqlHelper 클래스에 정의되어 있으며 매크로 내에서는 별칭으로 제공되는 SQL 을 사용할 수 있습니다.

* `IN` 메서드

  `IN` 메서드는 SQL `IN` 절(clause)을 생성해 주는 헬퍼 메서드 입니다. 매개변수가 배열과 같이 `IEnumerable` 인터페이스를 지원하는 경우 여러 항목으로 구성된 `IN` 절이 생성되며 그렇지 않다면 단일 항목의 `IN` 절이 생성됩니다.

  `SQL.IN` 메서드는 생성되는 `IN` 절의 값들을 따옴표로 묶습니다. 정수나 실수와 같이 따옴표가 필요없는 경우 `IN` 메서드의 두 번째 매개변수에 `false` 값을 지정해 주면됩니다.

  ```cs
  var parameters = new { Categories = new int[] { 1, 2, 3 }};
  this.DbAccess.ExecuteQueryDataSet("File.Qid", parameters);
  ```

  ```cs
  var str1 = SQL.IN(env.Args.Categories);  // IN ('1', '2', '3') 반환
  var str2 = SQL.IN(env.Args.Categories, true);  // IN (1, 2, 3) 반환
  ```

* `SET` 메서드

  `SET` 메서드는 SQL `UPDATE` 문장의 `SET` 절을 생성해 주는 헬퍼 메서드입니다. SET 메서는 `IDictionary` 를 매개변수로 사용하여 `IDictionary` 의 키를 컬럼이름으로 사용하며 `IDictionary` 의 값을 SET의 값으로 사용합니다. Fox Data Service 환경에서 `env.Args` 속성은 항상 `IDictionary` 타입이므로 `env.Args` 속성을 사용하면 SET 절을 손쉽게 구성할 수 있습니다.

  ```cs
  var dic = new Dictionary<string, object>();
  dic.Add("p1", "value1");
  dic.Add("p2", "value2");
  dic.Add("p3", 3);
  var str1 = SQL.SET(dic);
  // SET p1 = 'value1', p2 = 'value2', p3 = '3' 반환
  ```

  :warning: 이 SET 메서드는 SQL Injection에 취약할 수 있습니다. 사용 시 주의가 필요합니다.

  `SET` 메서드는 두번째 매개변수로 Fox Query 매개변수 컬렉션(즉, `env.Params` 속성)을 사용할 수 있습니다. Fox Query 매개변수 컬렉션이 사용되면 `SET` 메서드는 Fox Query 매개변수 컬렉션에 매개변수가 존재하지 않는 경우 SQL 매개변수를 추가하여 SQL Injection에 대해 안전한 SQL 문장을 생성해 줍니다.

  ```cs
  var str1 = SQL.SET(env.Args, env.Params);
  // SET p1 = #p1#, p2 = #p2#, p3 = #p3# 반환
  // 최종적으로 SET p1 = @p1, p2 = @p2, p3 = @p3 문장으로 변환
  ```

## 고려 사항

Fox Query의 동적 쿼리 기능은 다른 쿼리 매퍼들과 달리 C# 스크립트 코드를 사용하므로 여러 가지 장점과 더불어 주의해야 할 사항과 단점도 가지고 있습니다. Fox Query 매크로 기능 사용 시 고려해야할 사항들은 다음과 같습니다.

### 스크립트 환경

Fox Query 의 매크로는 별도의 C# 스크립트 엔진을 사용하여 수행됩니다. 따라서 일반적인 C# 기반의 코드 개발과는 다른 문제 해결 접근 방식을 사용해야 합니다.

* 런타임 컴파일

  스크립트 환경은 별도의 컴파일 과정이 없이 런타임에 코드가 컴파일되고 캐시 됩니다. 따라서 코드의 문법 오류도 최초 수행 시점에 가서야 문법 오류가 발견됩니다. 예를 들어, 오타에 의해 선언되지 않은 변수가 사용되었다고 가정해 봅시다.

  ```cs
  var categoryId = env.Args.CategoryId;
  if (DBNull.Value.Equals(categoryID)) {  // categoryID: 선언되지 않은 변수!
    return null;
  }
  ```

  일반적인 C# 개발 환경이라면 위 코드는 컴파일 도중 오류가 발생하며 원인이 무엇인지 즉시 알 수 있습니다. 하지만 스크립트 환경에서는 Fox Query 가 수행될 때 다음과 같은 런타임 오류가 발생합니다.

  ```log
  TheOne.Data.Extensions.Script.FoxQueryScriptException: 스크립트[WHERE] 컴파일 오류: 개체 참조가 개체의 인스턴스로 설정되지 않았습니다. ---> System.NullReferenceException: 개체 참조가 개체의 인스턴스로 설정되지 않았습니다.
  ```

  위 오류 메시지는 JavaScript 와 같은 스크립트 환경과 비슷하게 선언되지 않은 변수가 null 값을 가지게 되며 이로 인해 `NullReferenceException` 예외가 발생하게 됩니다.

* 오류 원인 파악

  앞서 예제에서 더 문제되는 부분이 존재합니다. 예제에서 발생한 오류 메시지는 컴파일 타임 오류 메시지와 달리 오류가 발생한 위치를 알려주지 않는다는 것입니다.
  
  :information_source: 이 오류 메시지는 Fox Query 가 C# 스크립트 엔진으로 사용하는 Z.Expressions.Eval 라이브러리의 오류 메시지이므로 당분간 수정될 가능성이 높지 않습니다.

  따라서, 예외 발생 시 다양한 가능성을 고려하고 예외 발생 원인을 찾아야 합니다.

* 런타임 바인딩

  스크립트 환경은 컴파일러에서 제공하는 일부 기능을 사용하지 못합니다. 예를 들어 == 연산자는 컴파일러에 의해 Equals 메서드 호출로 컴파일 되기도 합니다. 하지만 스크립트는 런타임 바인딩을 통해 == 연산자가 선택되기 때문에 문제를 유발할 수 있습니다.

  ```cs
  if (env.Args.p1 == DBNull.Value)  // 런타임 오류 발생.
  {
    // DBNull.Value 일 경우 처리
  }
  ```

  위 스크립트 코드는 일반적인 C# 컴파일 환경에서는 문제를 유발하지 않지만 스크립트 환경에서는 p1 매개변수 값에 따라 런타임 오류를 발생합니다. 이는 런타임 바인더가 p1 값의 타입(정수, 문자열 등)에 따라서 == 연산자 메서드를 찾아 호출하기 때문입니다. 일반적으로 정수, 문자열에 대한 == 연산자는 DBNull.Value 에 대한 오버로드를 제공하지 않기 때문에 런타임 오류가 발생합니다. 따라서 == 연산자 대신 Equals 메서드를 사용해야 합니다.

  ```cs
  if (DBNull.Value.Equals(env.Args.p1) == true)
  {
    // DBNull.Value 일 경우 처리
  }
  ```

### Fox Data Service 환경

Fox Query가 동적 쿼리를 지원하는 가장 큰 이유는 Fox Data Service 환경에서 간단한 조건절 제어를 위해서도 비즈니스 로직 컴포넌트를 작성해야 하는 불합리성을 제거하기 위함입니다. 매개변수에 따라 간단하게 `WHERE` 절을 제어하는 수준이라면 비즈니스 로직 컴포넌트를 작성하지 않고 Fox Query 매크로를 사용하여 Fox Data Service 를 활용할 수 있습니다.

Fox Data Service 환경에서 Fox Query 매크로를 사용할 때에는 주의할 사항이 존재합니다.

* `null` 값 변경

  Fox Data Service의 매개변수 인자값을 전달하는 `FoxServiceParameterCollection` 클래스(`Parameters` 속성)는 매개변수 값으로 `null` 이 사용되면 `DBNull.Value` 로 전환하는 기능을 제공합니다.

  :information_source: null 값 변환 기능은 `FoxServiceParameterCollection` 클래스의 `UseDBNull` 속성을 통해(기본값 `true`) 제어가 가능합니다.

  따라서 Fox Query 매크로에서 `env.Args` 속성 값에서 `null` 값 확인 대신 `DBNull.Value` 값을 확인해야 합니다.

  ```cs
  if (env.Args.p1 == null || DBNull.Value.Equals(env.Args.p1) == true)
  {
    // null 혹은 DBNull.Value 인 경우 처리 코드...
  }
  ```

---
