# NeoDEEX 4.5.6.1 HotFix 2021-01-19 (버전 4.5.6.3)

이 파일은 NeoDEEX 4.5.6.1 버전에 대한 HotFix (버전 4.5.6.3) 입니다.

## 주요 변경 사항

이 HotFix는 다음 문제를 해결합니다.

* Bug 776: FoxTextFileLogger가 ExculsiveLock이 true 일 때 Mutex 생성 시 오류가 발생함.

  이 오류는 IIS의 여러 AppPool이 Fox Biz Service를 사용하고 단일 로그 파일에 로그를 기록할 때, 로그 메시지가 섞이는 것을 방지하기 위해 ExculsiveLock을 true로 지정하는 경우 발생할 수 있습니다. ExclusiveLock 기능은 명명된 Mutex를 사용하여 동기화를 제공합니다만 한 AppPool이 서로 다른 계정을 사용하여 Mutex를 생성하는 동안 다른 AppPool이 생성된 Mutex에 접근하려 할 때 권한 오류(UnauthorizeException)가 발생할 수 있습니다. 이 패치는 이러한 문제가 발생하지 않도록 Mutex 생성 시 권한을 지정하여 공유가 가능하도록 합니다.

## 변경된 파일

* TheOne.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.3*
  * Size : *152,064*
  * Last write : *‎2021‎년 ‎1‎월 ‎19‎일 ‎화요일, ‏‎오후 4:13:20*

## 설치 방법

* 변경된 파일을 NeoDEEX 4.5 설치 폴더(일반적으로 C:\Program Files (x86)\NeoDEEX 4.5 입니다)의 Assemblies 폴더에 복사합니다. 기존 파일을 먼저 백업해 두길 권장합니다.

* 소스 제어를 통해 개발자에게 배포하는 경우, 소스 제어 폴더에 변경된 파일을 복사합니다.

* 이미 구성되고 배포된 업데이터 서버가 존재한다면 변경된 파일을 웹 어플리케이션의 bin 디렉터리에 복사합니다.

## 추가 정보

이 HotFix의 변경 사항은 다음 릴리스(4.5.7.0)에 포함됩니다.

---
