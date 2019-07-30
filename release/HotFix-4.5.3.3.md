# NeoDEEX 4.5.3.0 HotFix 2019-07-30 (버전 4.5.3.3)

이 파일은 NeoDEEX 4.5.3.0 버전에 대한 HotFix (버전 4.5.3.3) 입니다.

## 주요 변경 사항

이 HotFix는 다음 문제를 해결합니다.

* Updater 서비스에 상대 경로를 사용하면, 다운로드 기본 경로를 벗어나는 파일들을 다운로드 할 수 있음. (Bug 714)

> 주의: 4.5.3.3 버전을 적용하면 기존 업데이터 설정으로는 업데이터가 정상적으로 작동하지 않을 수 있습니다. Updater의 배포 기본 디렉터리의 하위 디렉터리에서만 다운로드가 허용되기 때문입니다.

## 문제 해결

### 패키지 폴더 개념

Updater에서 사용하는 패키지 디렉터리는 패키지 기본 디렉터리(baseDir), 패키지 디렉터리(packageDir), 패키지 하위 디렉터리(subDir)로 나누어 집니다.

패키지 기본 디렉터리(baseDir)는 패키지 다운로드를 위한 베이스 디렉터리로서 이 디렉터리의 하위에 여러 패키지 디렉터리(packageDir)가 존재합니다. 패키지 기본 디렉터리는 `web.config` 파일의 `<appSettings>` 항목에 `NeoDEEX_PackageBaseDirectory` 키를 통해 설정이 가능합니다. baseDir은 절대 경로 혹은 웹 어플리케이션 경로로부터 상대 경로로 지정이 가능합니다.

```xml
  <appSettings>
    <add key="NeoDEEX_PackageBaseDirectory" value="D:\Site\Deploy" />
  </appSettings>  
```

명시적으로 baseDir이 지정되지 않은 경우 웹 어플리케이션 경로가 baseDir로 설정됩니다.

패키지 디렉터리는 하나의 업데이터 웹 서비스를 통해 여러 어플리케이션 배포를 수행하기 위해서 존재하며 개별 어플리케이션 마다 패키지 디렉터리를 가질 수 있습니다. 따라서 패키지 디렉터리는 기본 디렉터리의 상대 경로로 표현됩니다. packageDir은 Updater 클라이언트가 매개변수로서 Updater 서버에 요청하며 `FoxUpdateSettings` 객체의 `PackageDirectory` 속성 값으로 패키지 디렉터리를 명시합니다.

다음과 같은 Updater 클라이언트 코드는 baseDir 폴더의 하위에 존재하는 `MyAppDeploy` 폴더를 packageDir로 설정하게 됩니다.

```cs
FoxUpdateSettings settings = new FoxUpdateSettings(SERVER_URL, localDir);
settings.PackageDirectory = "MyAppDeploy";
```

packateDir이 명시되지 않은 경우, 기본값은 `Deploy` 폴더로서 baseDir의 하위에 존재하는 `Deploy` 폴더가 됩니다.

패키지 하위 디렉터리(subDir)는 배포할 어플리케이션이 방대한 경우 배포를 나누어 할 수 있도록 하위 디렉터리 별로 구분할 수 있도록 해줍니다. 따라서 패키지 하위 디렉터리는 패키지 디렉터리(packageDir)로부터 상대 경로로 표현됩니다. subDir 역시 Updater 클라이언트가 매개변수로서 전달하며 생략된 경우 packageDir 과 동일하게 지정됩니다.

```txt
http://update.server.com/updater/download?packageDir=.&subDir=~/&filename=module.dll
```

Updater 클라이언트는 패키지 파일을 다운로드할 때 자동으로 subDir을 계산하게 되므로 프로그램적으로 subDir을 설정하는 경우는 없습니다.

### 구성 설정 변경

4.5.3.3 버전 이후부터 pakcageDir 이나 subDir을 통해 지정되는 경로가 baseDir의 하위 디렉터리가 아닌 경우 다운로드가 허용되지 않고 400 Bad Request 오류를 반환하게 됩니다.

따라서, 임의의 상대 경로를 허용하는 기존 Updater 설정을 그대로 사용하는 경우 Updater 서버가 400 Bad Request를 반환할 수 있습니다. 이러한 문제를 해결하기 위해서 다음 지침을 따르십시오.

1. `web.config` 파일의 `<appSettings>` 항목에 `NeoDEEX_PackageBaseDirectory` 키를 통해 명시적으로 baseDir을 지정하십시오.

    ```xml
    <appSettings>
        <add key="NeoDEEX_PackageBaseDirectory" value="D:\XX_Site\Deploy" />
    </appSettings>  
    ```

2. Updater 클라이언트 코드(대개 Program.cs)에서 packateDir 설정을 `web.config`에서 지정된 baseDir로 부터의 상대 경로로 수정하십시오.

    ```cs
    FoxUpdateSettings settings = new FoxUpdateSettings(SERVER_URL, localDir);
    settings.PackageDirectory = ".";
    ```

3. Updater 클라이언트를 빌드하고 다시 배포 하십시오. (Click-Once 배포) 이때 다음 문서를 참조하여 Click-Once 업데이트가 반드시 이루어지도록 조정하는 것이 좋습니다.

    [Click-Once 업데이트를 필수로 만들기](https://docs.microsoft.com/ko-kr/visualstudio/deployment/choosing-a-clickonce-update-strategy?view=vs-2015#making-updates-required)

## 변경된 파일

* NeoDEEX.Deployment.Web.Http.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.3.3*
  * Size : *12,288*
  * Last write : *2019-07-30 10:41 AM (GMT+9)*

## 설치 방법

* 변경된 파일을 NeoDEEX 4.5 설치 폴더(일반적으로 C:\Program Files (x86)\NeoDEEX 4.5 입니다)의 Assemblies 폴더에 복사합니다. 기존 파일을 먼저 백업해 두길 권장합니다.
* 이미 구성되고 배포된 업데이터 서버가 존재한다면 변경된 파일을 웹 어플리케이션의 bin 디렉터리에 복사합니다.

## 문제 해결 확인 방법

웹 브라우저에서 Updater Service의 download URL에 packageDir 및 subDir 매개변수에 패키지 기본 디렉터리를 벗어나는 상대 경로를 사용할 때 HTTP 400 BadRequest 오류가 발생해야 합니다.

## 추가 정보

이 HotFix의 변경 사항은 다음 릴리스(4.5.4.0)에 포함됩니다.

---
