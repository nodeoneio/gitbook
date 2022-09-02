# Cleos 기초

## 개요

cleos 는 로컬 노드의 nodeos, keosd 가 제공하는 REST API 와 통신할 수 있는 명령줄 인터페이스(CLI) 입니다. 스마트 컨트랙트를 개발할 때 cleos 를 사용하여 배포하고 테스트 할 수 있습니다.

cleos 는 Leap 소프트웨어의 일부분이기 때문에 Leap 를 설치할 때 같이 설치됩니다.

## Cleos 사용하기

cleos 를 사용하려면 nodeos 를 설정할 때 IP 주소와 포트 번호로 구성된 엔드포인트를 작성해야 합니다. 또한  nodeos 를 시작할 때 `eosio::chain_api_plugin` 이 로딩되어 있어야 합니다. 이렇게 구성해야 nodeos 가 cleos 로 부터 수신되는 RPC 요청에 응답할 수 있습니다.

cleos 는 기본적으로 로컬 노드의 HTTP RPC API 엔드포인트와 통신하기 때문에 curl 명령어로 HTTP API 를 호출해도 동일한 결과를 얻을 수 있습니다. 예를 들어 로컬 노드의 HTTP 엔드포인트 포트 번호가 8888 로 설정된 경우 아래 두 명령은 같은 결과를 보여줍니다.

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

cleos 명령 목록을 확인하려면 다음과 같이 입력합니다.

```
cleos --help
```

cleos 의 하위 명령에 대한 도움말도 확인할 수 있습니다. 예를 들어 `cleos create` 하위 명령에 대한 도움말을 확인하려면 다음과 같이 입력합니다.

```
cleos create --help
```

이보다 하위 명령도 같은 방식으로 확인할 수 있습니다.

help 옵션 외에도 EOSIO 개발자 포털에서 [Cleos 명령어 레퍼런스](https://developers.eos.io/manuals/eos/latest/cleos/command-reference/index)를 확인할 수 있습니다. 다만 현재 지원이 끊긴 EOSIO 의 레퍼런스 문서이기 때문에 Leap 에서 사용하는 명령어와 차이가 있을 수 있습니다.

## Cleos 명령 예제

아래는 cleos 를 사용하여 mywallet 이라는 이름의 지갑을 만들고 이 지갑의 패스워드를 콘솔에 표시하는 명령입니다.

```bash
cleos wallet create -n mywallet --to-console
```

```
Creating wallet: mywallet
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JbF34UdA193Eps1bjrWVJRaNMt1VKddLn4Dx6SPVTfMDRnMBWN"
```
