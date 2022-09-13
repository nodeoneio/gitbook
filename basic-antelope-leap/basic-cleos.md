# Cleos 기초

## 개요

`cleos` 는 로컬 노드의 nodeos, keosd 가 제공하는 REST API 와 통신할 수 있는 명령줄 인터페이스(CLI) 입니다. 스마트 컨트랙트를 개발할 때 `cleos` 를 사용하여 배포하고 테스트 할 수 있습니다.

`cleos` 는 Leap 소프트웨어의 일부분이기 때문에 Leap 를 설치할 때 같이 설치됩니다.

## Cleos 사용하기

`cleos` 를 사용하려면 `nodeos` 를 설정할 때 IP 주소와 포트 번호로 구성된 엔드포인트를 작성해야 합니다. 또한  `nodeos` 를 시작할 때 `eosio::chain_api_plugin` 이 로딩되어 있어야 합니다. 이렇게 구성해야 `nodeos` 가 `cleos` 로 부터 수신되는 RPC 요청에 응답할 수 있습니다.

`cleos` 는 기본적으로 로컬 노드의 HTTP RPC API 엔드포인트와 통신하기 때문에 `curl` 명령어로 HTTP API 를 호출해도 동일한 결과를 얻을 수 있습니다. 예를 들어 로컬 노드의 HTTP 엔드포인트 포트 번호가 8888 로 설정된 경우 아래 두 명령은 같은 결과를 보여줍니다.

```
curl http://localhost:8888/v1/chain/get_info
```

```
cleos get info
```

만약 원격 노드의 엔드포인트와 통신하고자 하는 경우라면 다음과 같이 -u 옵션과 원격 엔드포인트 주소를 지정하고 실행하면 됩니다.

```
cleos -u http://<remote_api_address:port> get info
```

## Cleos 명령

`cleos` 명령 목록을 확인하려면 다음과 같이 입력합니다.

```
cleos --help
```

`cleos` 의 하위 명령에 대한 도움말도 확인할 수 있습니다. 예를 들어 `cleos create` 하위 명령에 대한 도움말을 확인하려면 다음과 같이 입력합니다.

```
cleos create --help
```

이보다 하위 명령도 같은 방식으로 확인할 수 있습니다.

help 옵션 외에도 EOSIO 개발자 포털에서 [Cleos 명령어 레퍼런스](https://developers.eos.io/manuals/eos/latest/cleos/command-reference/index)를 확인할 수 있습니다. 다만 현재 지원이 끊긴 EOSIO 의 레퍼런스 문서이기 때문에 Leap 에서 사용하는 명령어와 차이가 있을 수 있습니다.

## Cleos 명령 예제

아래는 `cleos` 를 사용하여 mywallet 이라는 이름의 지갑을 만들고 이 지갑의 패스워드를 콘솔에 표시하는 명령입니다.

```bash
cleos wallet create -n mywallet --to-console
```

```
Creating wallet: mywallet
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JbF34UdA193Eps1bjrWVJRaNMt1VKddLn4Dx6SPVTfMDRnMBWN"
```

## 도메인 소켓(IPC) vs HTTP RPC

`cleos` 와 `keosd` 에 접근할 수 있는 방법은 HTTP RPC 외에도 도메인 소켓을 이용하는 방법이 있습니다. 도메인 소켓을 사용하면 여러 이점이 있습니다. HTTP RPC 는 외부 인터넷이나 LAN/WAN 으로 접근 할 수 있어 자칫 치명적인 기능의 접근 경로가 유출될 수도 있고, CORS 와 같은 여러가지 공격 경로에도 노출될 수도 있지만, 도메인 소켓은 내부 프로세스간의 통신 용도로만 사용하도록 의도된 방식이기 때문에 상대적으로 안전합니다.

## 트랜잭션 제출 후 나타나는 "transaction executed locally, but may not be confirmed by the network yet" 메시지의 의미

`cleos` 가 제출한 트랜잭션이 이 트랜잭션을 전달 받은 로컬 `nodeos` 인스턴스에 의해 성공적으로 수락되고 실행되었음을 의미합니다.&#x20;

다만, 이 `nodeos` 인스턴스가 제출받은 트랜잭션을 P2P 프로토콜을 통해 다른 노드로 릴레이 하더라도, 이러한 노드들도 이 트랜잭션을 수락하거나 실행할 거라는 보장은 없습니다. 또한 이 시점에서 트랜잭션이 BP에 의해 수락되고 실행된 뒤 블록체인의 유효한 블록에 포함될 것이라는 보장도 할 수 없습니다.&#x20;

따라서 트랜잭션이 비가역성 블록에 포함되었는지 더 확실하게 확인해야 하는 경우, 추가적인 작업을 수행하여 트랜잭션이 블록체인의 비가역성 블록에 있는지 모니터링 해야 합니다.&#x20;
