# 테스트넷에 연결된 노드 만들기(작업중)

## 개요

스마트 컨트랙트를 개발하고 테스트 할 때 로컬 노드에서도 할 수 있지만 메인넷에 배포하기 앞서 메인넷과 같은 환경으로 구성된 공식 테스트넷에서도 테스트 해 볼 필요가 있습니다. 이 단원에서는 테스트넷에 연결된 로컬 노드를 만드는 방법을 알아보겠습니다.

## 테스트넷

EOS 의 공식 테스트넷은 Jungle Testnet v4(이하 정글넷) 와 Kylin Testnet(이하 kylin 넷) 이 있습니다.

이 단원에서는 정글넷에 연결된 노드를 만들어 보겠습니다. 이 노드는 블록을 생성하지 않으며 정글넷에 p2p로 연결된 피어 노드와 동기화된 로컬 노드를 만들어 보는 것이 목적입니다.

이미 구성된 블록체인 네트워크와 동기화된 노드를 시작하는 방법은 다음과 같습니다.

* 스냅샷을 사용하여 노드 시작
* 블록 로그를 다운로드 받아 시작
* 제네시스 블록 부터 재생

스냅샷 다운로드



{% embed url="https://snapshots.eosnation.io/" %}

위 사이트에서 Leap 3.1 버전 전용 스냅샷인 **Jungle 4 Testnet - v6** 의 **latest** 파일을 로컬 노드로 다운로드 합니다. 스냅샷 파일은 zst 형식으로 압축되어 있으므로 unzstd 명령으로 압축을 풀어줘야 합니다.





제네시스 파일부터 리플레이 하기



이 노드는 블록을 생성하지도 않고 외부에 API 를&#x20;

config.ini 파일 설&#x20;

```
plugin = eosio::http_plugin
plugin = eosio::chain_api_plugin
http-server-address = 0.0.0.0:8888
http-validate-host = false
http-threads = 6
access-control-allow-origin = *
access-control-allow-headers = Origin, X-Requested-With, Content-Type, Accept
http-max-response-time-ms = 100
enable-account-queries = true
verbose-http-errors = true
chain-state-db-size-mb = 16384

p2p-peer-address = jungle.edenia.cloud:9877
p2p-peer-address = peer.jungle4.alohaeos.com:9876
p2p-peer-address = jungle4.eosphere.io:9876
p2p-peer-address = jungle4.seed.eosnation.io:9876
```

