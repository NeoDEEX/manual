# NeoDEEX 4.5.3.0 HotFix 2019-04-05 (버전 4.5.3.2)

이 파일은 NeoDEEX 4.5.3.0 버전에 대한 HotFix (버전 4.5.3.2) 입니다.

## 주요 변경 사항

이 HotFix는 다음 문제를 해결합니다.

* Updater Service(Web API)에 대해 파일 다운로드 요청 시 절대 경로를 사용하면 서버로부터 임의의 파일이 다운로드 될 수 있음. (Bug 707)

## 변경된 파일

* NeoDEEX.Deployment.Web.Http.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.3.2*
  * Size : *11,776*
  * Last write : *2019-04-05 03:58 PM (GMT+9)*

## 설치 방법

* 변경된 파일을 NeoDEEX 4.5 설치 폴더(일반적으로 C:\Program Files (x86)\NeoDEEX 4.5 입니다)의 Assemblies 폴더에 복사합니다. 기존 파일을 먼저 백업해 두길 권장합니다.
* 이미 구성되고 배포된 업데이터 서버가 존재한다면 변경된 파일을 웹 어플리케이션의 bin 디렉터리에 복사합니다.

## 문제 해결 확인 방법

웹 브라우저에서 Updater Service의 download URL에 subDir 매개변수에 절대 경로를 사용하여 다운로드를 시도할 때 HTTP 400 BadRequest 오류가 발생해야 합니다.

## 추가 정보

이 HotFix의 변경 사항은 다음 릴리스(4.5.4.0)에 포함됩니다.

---
