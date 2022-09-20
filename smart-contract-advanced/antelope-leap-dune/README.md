---
description: Docker Utilities for Node Execution
---

# 컨테이너 기반 Antelope 노드 실행 도구 D.U.N.E

## 개요

ENF 에서 관리하는 블록체인 네트워크 기반 소프트웨어인 Antelope Leap 는 현재 Ubuntu Linux 버전 18.04 부터 22.04 까지만을 네이티브 환경으로 지원하고 있으며, 당분간 지원하는 플랫폼을 추가하지는 않을 예정이라 하였습니다.

따라서 Windows 나 Mac OS 와 같은 이종 OS , 또는 RHEL 과 같은 지원 대상 외 리눅스 배포판에서 Antelope 기반 블록체인 환경을 구성하려면 VM 이나 컨테이너 환경을 이용해야 합니다.

일일히 노드를 설정하는 작업은 번거롭기 때문에 ENF 는 Docker 컨테이너 환경에서 개발자들이 쉽게 Antelope 로컬 노드를 구성하고 스마트 컨트랙트를 테스트 할 수 있도록 D.U.N.E(Docker Utilities for Node Execution) 이라는 도구를 만들고 공개했습니다.

이 단원에서 D.U.N.E 를 사용하여 스마트 컨트랙트 개발 환경을 만들고 간단히 테스트 해 보겠습니다.

## 구성방법

먼저 D.U.N.E 의 실행 환경을 구성해 봅시다. 환경에 따라 설정 방법이 사소하게 다르지만 기본적으로 다음과 같은 순서로 설치가 진행됩니다.

1. Docker 인스톨
2. Python3 인스톨
3. D.U.N.E 을 로컬 환경에 저장하고 경로를 PATH 변수에 추가
4. `bootstrap` 파일을 실행하여 Docker 이미지 제작

### Docker 인스톨

먼저 Docker 를 인스톨 합니다. 다음 공식 웹 사이트에서 제공하는 Docker 인스톨 방법을 참조합니다.

{% embed url="https://docs.docker.com/engine/install/" %}

### Python3 인스톨

다음으로 Python3 을 설치합니다. 대부분의 모던 OS 에는 기본적으로 Python3 이 설치되어 있지만, 그래도 설치되어 있지 않거나, 설치되어 있더라도 버전이 낮은 경우에는 다음 웹 사이트에서 자산의 플랫폼에 맞는 Python3 을 설치합니다.

{% embed url="https://www.python.org/downloads/" %}

### D.U.N.E 다운로드 및 PATH 에 경로 추가

다음으로 아래 D.U.N.E 의 공식 Github 에서 소스 코드를 다운로드 받습니다. zip 파일 다운로드나 git clone 명령 등 원하는 방법으로 소스 코드를 로컬 머신에 저장합니다.

{% embed url="https://github.com/AntelopeIO/DUNE" %}

다음으로 PATH 환경변수에 저장한 D.U.N.E 의 소스 코드 경로를 추가합니다. Mac OS 나 리눅스는 다음 명령으로 경로를 추가할 수 있습니다.

```
$ echo "PATH=<LocationOfDUNE>:$PATH" >> ~/.bashrc
$ source ~/.bashrc
```

윈도우의 경우 다음 웹사이트를 참조하여 PATH 환경 변수에 D.U.N.E 의 소스 코드 경로를 추가합니다.&#x20;

{% embed url="https://rootblog.tistory.com/206" %}

### Docker 이미지 제작

이제 다운받은 D.U.N.E 의 루트 디렉토리로 가서 `bootstrap.sh`(윈도우의 경우 `bootstrap.bat`)을 실행합니다.

```
# Mac OS, Linux
<PathToDUNE>/DUNE$ ./bootstrap.sh

# Windows
C:\<PathToDUNE>\DUNE$ .\bootstrap.bat
```

이 명령은 Antelope 노드의 Docker 이미지를 작성합니다. 이 작업은 로컬 머신의 성능에 따라 수 분에서 수십 분 정도 소요됩니다. 작업이 진행되는 도중에 출력되는 경고는 리카르디안 컨트랙트가 없다는 경고로, 무시해도 문제 없습니다.

Docker 이미지 생성이 완료되면 다음 명령으로 확인해 볼 수 있습니다.

```
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
dune latest dacb47a5a67b 5 seconds ago 3.73GB
```

## D.U.N.E 을 시작하기에 앞서

이 유틸리티는 `nodeos, cleos, CDT` 을 쉽게 다루기 위하여 추상화 된 명령을 제공합니다. 그렇기 때문에 일부 명령은 원래 명령에 비하여 사용상의 제약이 많다고 느껴질 수 있습니다. 만약 어떤 명령이 너무 제한적인 것 같다면, `dune --` 키워드 뒤에 필요로 하는 일반 `nodeos, cleos, CDT` 명령 또는 OS 명령을 붙여 실행 할 수 있습니다.

예를 들어 `cleos get info` 명령을 수행하려면 다음과 같이 입력합니다.

```
$ dune -- cleos get info
{
“server_version”: “5e9b0a28”,
“chain_id”: “8a34ec7df1b8cd06ff4a8abbaa7cc50300823350cadc59ab296cb00d104d2b8f”,
“head_block_num”: 1091,
“last_irreversible_block_num”: 1090,
“last_irreversible_block_id”: “00000442673382dca68abe469078912a860e408bd8efecbc1568ebc6b7dbc13f”,
“head_block_id”: “00000443b71711c7c4260bd9df7597ebf070ff4572719c62a78d9f83d3a2162f”,
“head_block_time”: “2022–07–14T16:55:59.500”,
...
```

컨테이너가 아직 생성되지 않은 경우 D.U.N.E 으로 아무 명령이나 실행하면 자동으로 생성됩니다. D.U.N.E 이 정상적으로 동작하는 상황이라면 `start-container` 명령을 꼭 실행할 필요는 없습니다.

개발자용 지갑은 자동으로 생성되며 항상 잠금 해제 상태이기 때문에, 잠금을 해제하도록 요청하는 명령은 따로 없습니다. 만약 `--` 옵션을 사용하여 어떤 `cleos wallet` 이나 `keosd` 명령을 실행한 뒤 지갑이 잠기게 된다면 D.U.N.E 의 wallet 명령 중 아무거나 하나를 실행하여 지갑을 잠금 해제할 수 있습니다.

또한 스마트 컨트랙트를 계정에 배포하면 해당 계정에 자동으로 `code` 권한이 추가됩니다.

현재 작업하고 있는 드라이브 또는 디렉토리는 노드를 실행하는 컨테이너 내부 디렉토리와 매핑되며, 이 디렉토리에는 `/host` 접두사가 붙습니다. 예를 들어 Windows 에서는 `/host/Users/<name>/<path>` 와 같은 경로가 되고, Linux 및 Mac OS 에서는 `/host/home/<name>/<path>` 가 됩니다.

## 노드 관리법

이제 간단하게 노드를 만들면서 `dune` 명령어를 실행해 보겠습니다. 대부분의 `dune` 명령어는 적어도 하나 이상의 노드가 동작하고 있어야 실행할 수 있습니다.

다음과 같이 `test_node` 라는 이름의 노드를 만들어 봅시다.

```
$ dune --start test_node
```

이 명령으로 새로운 Antelope 노드를 만들고 시작할 수 있습니다.

만약 노드에 다른 포트번호나 옵션을 주고 싶다면 커스텀 `config.ini` 를 만든 다음, 다음과 같이 노드 시작 시점에 지정 할 수 있습니다.

```
$ dune --start test_node <path-to-config>/config.ini
```

생성한 노드 목록을 확인하려면 다음과 같이 입력합니다.

```
$ dune --list
Node Name | Active? | Running? | HTTP | P2P | SHiP
 — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — 
test_node | Y | Y | 127.0.0.1:8888 | 0.0.0.0:9876 | 127.0.0.1:8080
```

위 목록에서 현재 작성한 노드들의 정보와 사용중인 포트 번호, 그리고 노드가 동작중이라면 active/inactive 상태를 확인할 수 있습니다.

D.U.N.E 은 상태 기반으로 동작합니다. 따라서 원하는 노드를 active 노드로 설정하고 명령을 실행하면 그 명령은 active 노드에서 바로 실행됩니다.

새 노드 생성하고 그시 노드가 성공적으로 시작되면 해당 노드는 active 노드로 자동으로 전환 됩니다. 만약 특정 노드를 active 노드로 전환하고 싶다면 다음과 같이 입력합니다.&#x20;

```
$ dune --set-active <node_name>
```

이제 시작한 노드의 정보를 확인해 보겠습니다. 아래 명령은 `cleos get info` 명령과 같은 결과를 출력합니다.

```
$ dune --monitor
{
 “server_version”: “90ebab68”,
 “chain_id”: “8a34ec7df1b8cd06ff4a8abbaa7cc50300823350cadc59ab296cb00d104d2b8f”,
 “head_block_num”: 7365,
 “last_irreversible_block_num”: 7364,
 “last_irreversible_block_id”: “00001cc45c86e5e20c07cabf0f829d1faab37685a1ba40b3247637c8424de911”,
 “head_block_id”: “00001cc5a12fbbe0f8d3bb1a8613fca2c183ecee4addd14fe05eec2227758f89”,
 ...
```

동작중인 노드를 중지하려면 다음과 같이 입력합니다.

```
$ dune --stop test_node
```

노드를 완전히 제거하려면 다음과 같이 입력합니다.

```
$ dune --remove test_node
```

### 다중 노드 <a href="#732b" id="732b"></a>

여러 노드를 만들고 싶지만 포트가 충돌하는 경우, 현재 실행 중인 노드를 중지하고 새 노드를 만들거나, `config.ini` 를 사용하여 변경된 포트를 적용한 노드를 추가로 시작하면 됩니다.

여러 노드를 병렬로 시작한뒤 `config.ini` 설정 파일을 적절히 구성하면, 마치 EOS 메인넷 같이 동작하는, 보다 복잡한 노드 토폴로지를 가진 프라이빗 블록체인 네트워크를 만들 수도 있습니다. 다만, 이런 식의 다중 노드 구성은 현재 D.U.N.E 에서 지원하지 않습니다. 만약 필요하다면 [바이오스 부트 시퀀스 ](../../basic-antelope-leap/bios-boot-sequence.md)항목을 참조하여 직접 구성하실 수 있습니다.
