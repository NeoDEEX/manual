# NeoDEEX 4.5.6.1 HotFix 2021-02-24 (버전 4.5.6.5)

이 파일은 NeoDEEX 4.5.6.1 버전에 대한 HotFix (버전 4.5.6.5) 입니다.

## 주요 변경 사항

이 HotFix는 다음 문제를 해결합니다.

* Task 788: FoxTextFileLogger의 로깅 전환 기능 제거.

* Task 787: 오류 시 DbProfile이 Error 로그 수준으로 기록되도록 코드 수정.

* Task 786: 오류 시 DbProfile이 항상 기록되도록 코드 수정.

* Task 790: ClassId를 로거 이름으로 사용하여 FoxBizService 로깅하는 기능 추가.

* Task 791: QueryId를 로거 이름으로 사용하여 FoxDataService 로깅하는 기능 추가.

* Task 775: DB 연결 문자열 설정에 접근할 수 있는 기능 추가.

* Issue 792: FoxDatabaseFactory에서 데이터베이스 연결 이름을 못 찾는 경우 오류 메시지가 모호함.

* Issue 793: 예외로 Rollback된 트랜잭션을 Commit 할 때 발생하는 예외가 오류의 원인을 파악하기 어려움.

## 변경된 파일

* TheOne.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.5*
  * Size : *152,064*
  * Last write : *2021‎년 ‎2‎월 ‎24‎일 ‎수요일, ‏‎오전 10:34:38*

* TheOne.Data.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.5*
  * Size : *130,048*
  * Last write : *2021‎년 ‎2‎월 ‎24‎일 ‎수요일, ‏‎오전 10:34:38*

* TheOne.ServiceModel.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.5*
  * Size : *131,584*
  * Last write : *2021‎년 ‎2‎월 ‎24‎일 ‎수요일, ‏‎오전 10:34:42*

* TheOne.ServiceModel.Activation.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.5*
  * Size : *63,488*
  * Last write : *‎2021‎년 ‎2‎월 ‎24‎일 ‎수요일, ‏‎오전 10:34:46*

* TheOne.Transactions.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.5*
  * Size : *39,936*
  * Last write : *‎2021‎년 ‎2‎월 ‎24‎일 ‎수요일, ‏‎오전 10:34:40*

## 설치 방법

* 변경된 파일을 NeoDEEX 4.5 설치 폴더(일반적으로 C:\Program Files (x86)\NeoDEEX 4.5 입니다)의 Assemblies 폴더에 복사합니다. 기존 파일을 먼저 백업해 두길 권장합니다.

* 소스 제어를 통해 개발자에게 배포하는 경우, 소스 제어 폴더에 변경된 파일을 복사합니다.

* 이미 구성되고 배포된 업데이터 서버가 존재한다면 변경된 파일을 웹 어플리케이션의 bin 디렉터리에 복사합니다.

## 추가 정보

이 HotFix의 변경 사항은 다음 릴리스(4.5.7.0)에 포함됩니다.

---
