# CDT(Contract Development Toolkit) 개요와 설치 방법

## 개요

CDT(구 EOSIO.CDT)는 WASM(Web Assembly)용 툴체인으로 Antelope 플랫폼의 스마트 컨트랙트 개발에 사용하는 도구입니다. CDT 는 범용 Web Assembly 툴체인 이기도 하지만 Antelope 스마트 컨트랙트를 수월하게 작성할 수 있도록 Antelope 플랫폼에 보다 최적화되어 있습니다.&#x20;

CDT 툴체인은 Clang9 으로 구축되었으며, 이는 CDT 가 LLVM 의 최적화 및 분석 도구를 보유하고 있다는 의미이지만, 아직 WASM 타겟은 실험적인 것으로 간주되기 때문에 일부 최적화 도구는 사용하기에 불완전하거나 아예 사용할 수 없습니다.

이 단원은 CDT 3.0.0 기준으로 작성되었습니다.&#x20;

## CDT 바이너리 설치하기

CDT 는 데비안 패키지 형태로 제공됩니다. 다음의 CDT 릴리스 페이지에서 바이너리를 다운로드 받을 수 있습니다.

{% embed url="https://github.com/AntelopeIO/cdt/releases" %}

다운로드 받은 패키지를 다음과 같이 설치할 수 있습니다. 아래 예제는 cdt 3.0.0 을 다운로드 받아 설치하는 명령입니다.

```
wget https://github.com/AntelopeIO/cdt/releases/download/v3.0.0/cdt_3.0.0_amd64.deb
sudo apt install ./cdt_3.0.0_amd64.deb
```

정상적으로 설치되었는지 다음 명령으로 확인합니다.

```
$ cdt-cpp --version
cdt-cpp version 3.0.0
```

## CDT 소스코드를 컴파일 하여 설치하기

(TBD)
