# NeoDEEX 4.5.6.1 HotFix 2021-03-17 (버전 4.5.6.7)

이 파일은 NeoDEEX 4.5.6.1 버전에 대한 HotFix (버전 4.5.6.7) 입니다.

## 주요 변경 사항

이 HotFix는 다음 문제를 해결합니다.

* Bug 798: FoxUpdaterClient가 관리자 모드로 구동할 때 Win32Exception 예외가 발생함.
* Bug 799: FoxTransaction의 COM+ 호환 모드에서 SetAbort가 트랜잭션을 롤백하지 않음.

## 변경된 파일

* NeoDEEX.Deployment.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.7*
  * Size : *45,568*
  * Last write : *‎2021‎년 ‎3‎월 ‎17‎일 ‎수요일, ‏‎오후 3:58:30*

* TheOne.Transactions.4.5.dll
  * Assembly version : 4.5.0.0
  * File version : *4.5.6.7*
  * Size : *40,960*
  * Last write : *‎‎2021‎년 ‎3‎월 ‎17‎일 ‎수요일, ‏‎오후 3:58:30*

## 설치 방법

* 변경된 파일을 NeoDEEX 4.5 설치 폴더(일반적으로 C:\Program Files (x86)\NeoDEEX 4.5 입니다)의 Assemblies 폴더에 복사합니다. 기존 파일을 먼저 백업해 두길 권장합니다.

* 소스 제어를 통해 개발자에게 배포하는 경우, 소스 제어 폴더에 변경된 파일을 복사합니다.

* 이미 구성되고 배포된 업데이터 서버가 존재한다면 변경된 파일을 웹 어플리케이션의 bin 디렉터리에 복사합니다.

## 추가 정보

이 HotFix의 변경 사항은 다음 릴리스(4.5.7.0)에 포함됩니다.

---
