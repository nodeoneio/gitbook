# Net Plugin

## 개요

`net_plugin` 은 인증된 p2p 프로토콜을 제공하여 노드를 지속적으로 동기화 합니다.

## 사용법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::net_plugin
[options]

# command-line
nodeos ... --plugin eosio::net_plugin [options]
```

## Net Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::net_plugin 설정 옵션

  --p2p-listen-endpoint arg (=0.0.0.0:9876)
    수신되는 p2p 커넥션을 받기 위한 host:port 주소.
  --p2p-server-address arg
    이 노드를 식별하기 위한 외부에서 접근 가능한 host:port 주소.
    기본값은 p2p-listen-endpoint 주소와 같다.
  --p2p-peer-address arg
    연결할 피어 노드의 public endpoint. 
    필요에 따라 여러 p2p-peer-address 옵션을 사용하여 네트워크를 구성할 수 있다.
    구문: host:port[:<trx>|<blk>]
    'trx' 와 'blk' 옵션은 트랜잭션(trx) 또는 블록(blk) 만을 보낼 것인지를 나타낸다.
    예시:p2p.eos.io:9876
        p2p.trx.eos.io:9876:trx
        p2p.blk.eos.io:9876:blk
  --p2p-max-nodes-per-host arg (=1)
    단일 IP 주소를 사용하여 접속하는 클라이언트 노드의 최대 숫자.
  --p2p-accept-transactions arg (=1)
    p2p 네트워크레서 수신한 트랜잭션이 유효하다면 처리하고 릴레이 한다.
  --agent-name arg (="EOS Test Agent")
    다른 peer 에서 이 노드를 식별하기 위하여 설정하는 이름.
  --allowed-connection arg (=any)
    'any' or 'producers' or 'specified' or 'none'. 
    'specified' 의 경우, peer-key 가 최소한 1번은 지정되어야 한다.
    'producers'만 지정된 경우, peer-key 는 필요 없다. 
    'producers' 와 'specified' 는 같이 사용될 수 있다.
  --peer-key arg
    (선택적) 연결이 허용된 peer 의 public key. 여러 번 사용할 수 있다.
  --peer-private-key arg
    [PublicKey, WIF private key] 의 튜플. 여러 번 사용할 수 있다.
  --max-clients arg (=25)
    연결이 허용된 클라이언트의 최대 수. (0 = 제한 없음)
  --connection-cleanup-period arg (=30)
    dead 커넥션을 없애기 전에 대기하는 시간. (초 단위)
  --max-cleanup-time-msec arg (=10)
    cleanup 호출 당 최대 연결 cleanup 타임 (ms 단위)
  --p2p-dedup-cache-expire-time-sec arg (=10)
    중복 최적화(duplicate optimization)를 위해 트랜잭션을 추적하는 최대 시간.
  --net-threads arg (=2)
    net_plugin 스레드 풀에서 동작하는 worker 스레드 수.
  --sync-fetch-span arg (=100)
    피어 동기화 중에 상대 피어 노로 부터 수신하는 블록 덩어리(chunk) 안에 들어가는 블록 개수.
  --use-socket-read-watermark arg (=0)
    socket read watermark 최적화를 사용. (실험적 옵션)
  --peer-log-format arg (=["${_name}" ${_ip}:${_port}])
    peer 에 대한 메시지를 로깅할 때, peer 의 포맷을 나타내는데 사용되는 문자열.  변수는 ${<variable name>} 으로 이스케이프 된다.
    사용 가능한 변수:
     _name  self-reported name
     _id    self-reported ID (64 hex characters)
     _sid   _peer.id 의 처음 8문자
     _ip    peer 의 원격 IP 주소
     _port  peer 의 원격 port 번호
     _lip   peer 에 연결하는데 사용하는 로컬 IP 주소
     _lport peer 에 연결하는데 사용하는 로컬 port 번호
  --p2p-keepalive-interval-ms arg (=32000)
    피어 노드의 heartbeat keepalive 메시지 간격. (ms 단위)
```

## 의존성

없음
