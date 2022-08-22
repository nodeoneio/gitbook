---
description: nodeos 는 어떻게 동작하는가
---

# Nodeos 의 동작 방식

## 노드 데몬(Daemon)과 상태(State)

nodeos 는 Antelope 블록체인의 가장 중요한 구성 요소이며, 트랜잭션과 블록을 처리하고 블록체인이 합의를 준수하도록 합니다. nodoes 는 리눅스 데몬 프로그램이고 프로덕션 환경에서는 일반적으로 우분투 베어 메탈 서버에서 실행됩니다.&#x20;

nodeos 는 모듈 구조를 가지고 있으며, 설정 파일에 몇 가지 필수 모듈과 선택적 모듈을 사용하도록 지정할 수 있습니다.

설정 옵션은 명령줄이나 `config.ini` 파일에서 지정할 수 있지만 `--data-dir` 및 `--config-dir` 와 같은 몇 가지 옵션은 명령줄에서만 사용할 수 있습니다. 지원되는 모든 옵션을 확인하려면 `nodeos --help` 를 입력합니다. 이들 중 일부는 추가 모듈을 활성화하고 있으며, 일부 옵션은 해당 모듈이 활성화되었을 때만 관련이 있습니다.

블록체인 상태(state)는 노드 데이터의 핵심으로, `--data-dir`  에 지정된 데이터 디렉토리의 하위 폴더 `/state` 에 저장됩니다. 이는 파일에 매핑된 공유 메모리 세그먼트로 구성됩니다. 이 메모리 내에서 노드는 블록체인 계정, 스마트 컨트랙트 테이블 및 인덱스를 추적하는데 사용하는 다수의 이진 트리를 관리합니다.&#x20;

상태는 리눅스 커널에 의해 하나의 큰 메모리 세그먼트로 간주되며 커널은 스토리지에 대한 매핑을 유지하기 위해 최선을 다하고 있습니다. 각 트랜잭션에서 일부 데이터는 상태 메모리를 읽고 쓰며 커널에 의해 완전히 무작위 액세스로 이루어집니다.

상태가 정상적인 파일 시스템에 있는 경우 커널은 가능한 빠르게 모든 업데이트를 디스크 상의 파일과 동기화 하려고 합니다. 지속적으로 활동하는 블록체인의 경우 가능한 I/O 지연 시간(Laterncy)을 최소화해야 하기 때문에 보통 클라우드 서비스 등에서 제공하는 가상화된 스토리지(예: AWS EBS)는 고려사항에서 제외됩니다. 가상화된 스토리지는 CPU와 물리적 스토리지 사이의 네트워크 계층으로 인해 로컬 스토리지보다 긴 지연 시간이 발생하는 것을 피할 수 없기 때문입니다. HDD 의 경우는 속도가 지나치게 느립니다. 현재 대부분의 퍼블릭용 블록 체인에서 블록 체인이 업데이트를 따라잡으려면 로컬 서버에 직접 물리적으로 연결된 SSD 또는 NVME 가 필요합니다.

블록체인의 활동이 많아지면 많아질수록 상태 업데이트가 증가하여 SSD 수명에 심각한 영향을 줍니다. EOS나 WAX와 같은 블록체인은 1년 이내에 NVME 드라이브를 망가뜨릴 수도 있습니다. 또한 WAX와 같은 블록체인에서 NVME는 노드 성능의 병목 현상이 되고 있습니다.&#x20;

NVME의 성능 열화를 방지하기 위해 Leap 노드는 tmpfs 파일 시스템을 많이 사용합니다. 이는 메모리 내에 존재하는 파일 시스템으로, 모든 콘텐츠를 서버 RAM과 스왑 공간(Swap Space)에 보관합니다. 상태 메모리의 절반 정도는 거의 사용되지 않기 때문에 성능 저하 없이 스왑 아웃이 가능하므로 서버 RAM 이 상태 메모리의 절반 정도의 크기를 가지는 것도 가능합니다.&#x20;

또한 리눅스 커널은 스마트한 방식으로 메모리를 다룹니다. 메모리에는 중복된 데이터가 없고 공유 메모리 클라이언트는 메모리에서 데이터를 직접 읽습니다. 또한 스왑 관리자는 접근 빈도가 가장 낮은 페이지를 디스크 스토리지에 밀어넣어 자주 접근하는 페이지를 위한 공간을 더 많이 제공합니다. 그리하여 충분한 RAM과 노드 상태(tmpfs)가 있는 서버에서 스토리지 I/O는 보통 수준이며 SSD 수명에 크게 영향을 끼치지 않습니다.

## 노드의 역할

블록체인 네트워크는 보통 다양한 역할을 하는 다수의 노드로 구성됩니다. 여러 가지 역이 필요하다면 각각의 역할에 대하여 별도의 노드를 실행하는 것이 일반적인 룰 입니다. 노드는 나중에 다룰 p2p 프로토콜을 통해 서로간에 블록과 트랜잭션을 교환합니다.&#x20;

* 블록 생산자(BP) 노드: 이 노드들은 p2p 프로토콜을 통해 서명된 트랜잭션을 수신하고 블록에 배치합니다. 한 번에 하나의 노드만 블록을 생성하며, 모두 일정에 따라 블록에 서명합니다. BP 노드가 위치한 서버의 로컬 시계가 신뢰할 수 있는 타임 소스와 동기화되는 것도 중요합니다. 일반적으로 이 노드의 성능이 전체 네트워크 안정성에 큰 영향을 끼치기 때문에 외부 트래픽으로부터 잘 보호해야 합니다.&#x20;
* HTTP RPC 노드: 이 노드는 클라이언트에 HTTP RPC 인터페이스를 제공합니다. 설정하기에 따라 아무나 연결할 수 있는 퍼블릭 노드를 만들 수도 있고, 특정 백엔드 애플리케이션만이 사용하는 전용 노드도 만들 수 있습니다.&#x20;
* P2P 허브 노드: 다른 노드의 p2p 연결을 허용합니다. 일반적으로 누구나 자신의 노드를 네트워크에 연결할 수 있습니다. 그렇기에 개인 p2p 노드도 드물지 않습니다. 또한 p2p 인터페이스에서 추측성 트랜잭션(Speculative Transaction)을 허용하지 않는 노드도 있어 가능한 최소 지연으로 블록 전파를 용이하게 합니다. 이러한 노드는 일반적으로 네트워크 성능을 향상시키기 위해 블록 생산자 조직에서 내부적으로 사용합니다.&#x20;
* 상태 기록 노드: 이 노드는 `state_history_plugin` 이 활성화된 상태에서 실행됩니다. 이 플러그인은 다양한 기록 인덱싱 솔루션에 사용되는 데이터 형식으로 트랜잭션 추적 및 컨트랙트 테이블 업데이트를 내보냅니다. 일반적으로 이러한 노드는 RPC 또는 p2p 인터페이스를 퍼블릭 클라이언트에게 공개하지 않습니다.&#x20;
* 파이어호스 노드: 딥마인드 플러그인이 활성화된 상태에서 실행되며 트랜잭션 추적 및 메모리 업데이트를 내보낼 수 있는 또 다른 방법을 제공합니다.

## 노드 인터페이스

동작중인 nodeos 프로세스는 여러가지의 API 프로토콜을 사용하여 외부 시스템과 통신합니다.

### HTTP RPC

조금 오래되긴 했지만 [EOSIO 개발자 포털](https://developers.eos.io/welcome/latest/reference/nodeos-rpc-api-reference)은 RPC 엔드포인트에 대한 모든 레퍼런스를 제공합니다. 아래는 블록체인 개발자들에게 필요한 가장 필수적인 내용들만 나열한 것입니다.  노드에 `http-server-address` 옵션을 설정하면 TCP 포트에 표준 HTTP 인터페이스를 제공합니다. 노드에서 자체적으로 HTTPS 도 지원하지만 일반적으로 nginx 와 같은 별도의 프록시 솔루션으로 SSL 종단점을 설정하는 것이 더 효율적이기 때문에 추천하지는 않습니다.

* `/v1/chain/get_info` 는 해당 노드에서 바라본 블록체인의 상태를 보여줍니다. 표시된 내용 중 중요한 정보는 다음과 같습니다.
  * chain\_id 블록체인을 구분하기 위한 32바이트의 고유한 식별자입니다.&#x20;
  * head\__block\__num 이 노드가 인식하고 있는 가장 마지막 블록 번호입니다.
  * head\__block\__time UTC 타임존에서 가장 마지막 블록의 타임스탬프입니다.
  * last\__irreversible\_block\_num_ (LIB) 노드에서 바라본 블록체인의 최종성(Finality) 를 나타냅니다. 블록체인이 오동작하면 LIB 가 멈춘 채 변하지 않습니다. 일반적인 상황에서 LIB는 헤드블록보다 300-350 이내의 번호를 가집니다.
  * last\__irreversible\_block\_id_ 각 블록은 자신이 가지고 있는 데이터 정보를 sha256 으로 해싱한 32바이트의 식별자 값을 가집니다. 트랜잭션을 보내기 위해서는 클라이언트는 유효한 블록 ID를 가르킬 필요가 있는데, LIB 의 식별자는 포크되어 취소될 염려가 없기 때문에 가장 신뢰할 만한 식별자입니다.
* `/v1/chain/get_block` 은 블록의 번호나 ID 값을 받아, 해당 블록의 정보를 JSON 형식으로 보여줍니다. 이는 트랜잭션 입력만 보여주며 트랜잭션 추적은 보여주지 않습니다. 자세한 내용은 [트랜잭션 라이프 사이클 챕터](https://cc32d9.gitbook.io/eosio-smart-contract-developers-handbook/how-eosio-works/life-cycle-of-a-transaction) 에서 찾아볼 수 있습니다.
* `/v1/chain/push_transaction` 과 `/v1/chain/send_transaction` 은 추가적인 검증 및 브로드캐스트를 위해 트랜잭션을 수락하는, 동일한 작업을 수행합니다. 두 API 의 다른점이 있다면 결과값인데, push\_transaction 계층 객체(hierarchical object)에서 추적을 반환하는 반면, 더 최신인 send\_transaction 은 추적을 일괄적인 목록으로 반환합니다. 또한 send\_transaction 은 계층형 객체를 준비할 필요가 없기 때문에 더 빨리 실행됩니다.&#x20;
* `/v1/chain/get_table_rows` 를 통해 스마트 컨트랙트 테이블에 질의하고 원하는 row 의 데이터를 수신할 수 있습니다. API는 특정한 Primary Key 또는 Secondary Key 의 값 범위를 받을 수 있으며 디코딩에 사용하는 컨트랙트 ABI를 사용하여 데이터를 역직렬화 한 형태로 반환합니다. 출력 결과 길이는 응답 시간에 따라 제한이 걸리므로 1,000개의 행을 요청해도 고작 수십 개의 행을 받는 경우도 있을 수 있습니다. 응답 결과에는 "more" 및 "next\_key" 매개 변수가 포함되어 있기 때문에, 이를 사용하여 클라이언트가 테이블로부터 더 많은 데이터를 가져올 수 있습니다.

### P2P 인터페이스

P2P 프로토콜은 노드들이 서로 필요한 모든 정보를 교환하기 위해 사용됩다. 여기에는 제어 메시지뿐만 아니라 블록 및 투기적(speculative) 트랜잭션이 포함됩니다. 노드는 p2p-listen-endpoint 설정 옵션에서 지정한 TCP 포트에서 P2P 연결을 허용합니다. 또한 p2p-peer-address 옵션에 나열된 다른 노드와도 연결됩니다. 통신 프로토콜은 대칭적 이기 때문에 누가 누구에게 연결되었는지는 중요하지 않으며 연결된 노드의 쌍은 동일한 방식으로 통신합니다. 즉, 노드가 들어오는 p2p 연결만 가지고 있든 나가는 p2p 연결만 가지고 있든 연결만 되면 서로간에 필요한 모든 데이터를 주고받는다는 의미입니다.

### 상태 이력 API (State History API)

어떤 노드는 컨트랙트 메모리의 모든 트랜잭션의 상세나 변경 사항을 모으고 내보내기 위해 `state_history_plugin (SHiP)`을 사용하기도 합니다. 이 API는 raw 웹소켓 프로토콜 사용하며 데이터는 직렬화된 형태로 내보내집니다. 상태 이력을 읽고 디코딩할 수 있는 여러가지 [오픈소스 라이브러리와 도구](https://cc32d9.medium.com/history-and-notifications-in-eosio-blockchain-8255194af93)들이 있습니다.