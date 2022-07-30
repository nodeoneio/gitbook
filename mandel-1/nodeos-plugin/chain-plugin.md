# Chain Plugin

## 개요

`chain_plugin` 은 Mandel 노드 상의 블록체인 데이터를 처리하고 통합하는데 필수적인 어 플러그인 입니다.

## 설정법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::chain_plugin
[options]

# command-line
nodeos ... --plugin eosio::chain_plugin [operations] [options]
```

## Chain Plugin 명령줄 전용 옵션

다음 옵션들은 명령줄에서 nodeos 를 실행할 때에만 사용할 수 있습니다.

```
eosio::chain_plugin 의 명령줄 옵션

  --genesis-json arg                    체인의 Genesis 파일 이름 및 경로.
  --genesis-timestamp arg               Genesis 파일의 초기 타임스탬프를 덮어씌운다.
  --print-genesis-json                  blocks.log 에서 genesis_state 를 추출해서 
                                        JSON 형태로 콘솔으로 출력한 뒤 빠져나간다.
  --extract-genesis-json arg            blocks.log 에서 genesis_state 를 추출해서 
                                        지정한 파일에 JSON 형태로 출력한 뒤 빠져나간다.
  --print-build-info                    빌드 환경 정보를 JSON 형태로 콘솔에 출력하고 빠져나간다.
  --extract-build-info arg              빌드 환경 정보를 추출하여 JSON 형태로 지정한 파일에 
                                        출력한 뒤 빠져나간다.
  --force-all-checks                    블록을 재생하는 동안 유효성 검사를 건너뛰지 않는다. 
                                        신뢰할 수 없는 출처로부터 블록을 재생해야 할 때 유용함.
  --disable-replay-opts                 리플레이 최적화 옵션을 사용하지 않는다.
  --replay-blockchain                   체인 상태 데이터베이스를 클리어하고 모든 블럭을 다시 
                                        재생한다.
  --hard-replay-blockchain              체인 상태 데이터베이스를 클리어하고 블록 로그에서 가능한 
                                        많은 블록을 복구한 후, 다시 이 블록을 재생한다.
  --delete-all-blocks                   모든 블록 로그와 체인 state 데이터베이스를 삭제한다.
  --truncate-at-block arg (=0)          0이 아닌 숫자를 입력했다면 그 숫자에 해당하는 블록.                                       
                                        해당 블록에 다다랐을 때 hard replay 와 블록 로그 
                                        복구를 중지한다.
  --terminate-at-block arg (=0)         0이 아닌 숫자를 입력했다면 그 숫자에 해당하는 블록 번호에 
                                        다다랐을 때 종료한다.
  --snapshot arg                        읽어들일 스냅샷 파일명.
```

## Chain Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::chain_plugin 설정 옵션들

  --blocks-dir arg (="blocks")
    blocks 디렉토리의 위치. (절대 경로나 data 디렉토리로 부터의 상대 경로)
  --protocol-features-dir arg (="protocol_features")
    protocol_features 디렉토리의 위치. (절대 경로나 data 디렉토리로 부터의 상대 경로)
  --checkpoint arg          
    체크포인트로 사용될 [BLOCK_NUM,BLOCK_ID] 의 쌍.
  --wasm-runtime runtime(=eos-vm)           
    WASM 인터프리터 런타임 설정.(현재 eos-vm 만 가능)
  --profile-account arg               
    프로파일 될 code 를 가지고 있는 계정 이름.
  --abi-serializer-max-time-ms arg (=15)
    ABI 직렬화에 허용되는 최대 시간 (ms 단위).
  --chain-state-db-size-mb arg (=1024)
    체인 상태 데이터베이스의 최대 크기 (MiB 단위).
  --chain-state-db-guard-size-mb arg (=128)
    체인 상태 데이터베이스에 남아 있는 여유 공간이 지정된 크기 아래로 내려가면 노드를 안전하게 종료한다.
    (MiB 단위)
  --signature-cpu-billable-pct arg (=50)
    실제 청구될 signature recovery cpu 의 백분율. 숫자 자체가 백분율이다.
    예를들어 50 이면 50% 를 의미.
  --chain-threads arg (=2)
    컨트롤러 스레드 풀에서 동작하는 worker 스레드의 수.
  --contracts-console
    스마트 컨트랙트의 아웃풋을 콘솔에 출력.
  --deep-mind
    자세한 체인 가동 정보를 출력.
  --actor-whitelist arg
    actor 화이트리스트에 추가된 계정. (여러 번 지정할 수 있다.)
  --actor-blacklist arg
    actor 블랙리스트에 추가된 계정. (여러 번 지정할 수 있다.)
  --contract-whitelist arg
    Contract 화이트리스트에 추가된 contract 계정. (여러 번 지정할 수 있다.)
  --contract-blacklist arg
    Contract 블랙리스트에 추가된 contract 어카운트 (여러 번 지정할 수 있다.)
  --action-blacklist arg
    Action 블랙리스트에 추가된 Action(code::action 의 형태) (여러 번 지정할 수 있다.)
  --key-blacklist arg
    사용하지 말아야 할 키의 블랙리스트에 추가된 공개키. (여러 번 지정할 수 있다.)
  --sender-bypass-whiteblacklist arg
    이 목록에 있는 계정에서 보낸 지연된 트랜잭션에는 주관적인 화이트리스트/블랙리스트 검사가 적용되지 않는다.
    (여러 번 지정할 수 있다.)
  --read-mode arg (=speculative)
    데이터베이스의 읽기 모드 ("speculative", "head", "read-only", "irreversible").
    - "speculative" 모드 - 데이터베이스에는 헤드 블록까지의 변경사항과 아직 블록체인에 포함되지 않은 
      트랜잭션의 변경사항이 포함되어 있다.
    - "head" 모드 - 데이터베이스에는 헤드 블록까지의 변경사항이 포함되어 있다.
    - "read-only" 모드 -(Deprecated: p2p-accept-transaction, api-accept-transaction 
      참조) 데이터베이스에는 마지막 비가역성 블록까지만 블록체인 트랜잭션에 의해 수행된 변경사항이 
      포함되어 있다. P2P 네트워크에서 받은 트랜잭션은 릴레이되지 않으며 트랜잭션을 체인 API에 
      푸시할 수 없다.
    - "irreversible" 모드 - 데이터베이스에는 마지막 비가역성 블록까지만 블록체인 트랜잭션에 의해 
      수행된 변경사항이 포함되어 있다. P2P 네트워크에서 받은 트랜잭션은 릴레이되지 않으며 트랜잭션을 
      체인 API에 푸시할 수 없다.
  --api-accept-transactions (=1)        
    API 트랜잭션이 유효하다면 처리 후 릴레이 된다.
  --validation-mode arg (=full)         
    체인 검증 모드 ("full" 또는 "light").
    - "full" 모드 - 들어오는 모든 블록이 완전하게 검증됨.
    - "light" 모드 - 들어오는 모든 블록 헤더가 완전히 검증됨.
    이렇게 검증된 블록의 트랜잭션들은 신뢰할 수 있다.
  --disable-ram-billing-notify-checks
    Contract 가 통지 핸들러(notification handler)의 컨텍스트 내에서 다른 계정에 더 많은 RAM을 
    청구하는 경우, 임의로 트랜잭션을 실패시키는 검사를 사용하지 않는다.
    (예를 들어 receiver 가 액션의 code 계정이 아닌 경우)
  --maximum-variable-signature-length arg (=16384) 
    임의로 가변 길이 서명 내의 가변 컴포넌트의 최대 길이를 설정한이 크기로 제한한다.
  --trusted-producer arg
    서명된 블록 헤더가 완전하게 검증된 블록의 트랜잭션도 신뢰할 수 있는 블록 작성자를 나타낸다.
  --database-map-mode arg (=mapped)
    데이터베이스 매핑 모드 ("mapped", "heap", or "locked").
    - "mapped" 모드 - 데이터베이스는 파일 형태로 메모리 매핑된다.
    - "heap" 모드 - 데이터베이스는 스왑 가능한 메모리에 미리 로드되며, 가능하면 
      거대 페이지(huge page) 를 사용한다.
    - "locked" 모드 - 데이터베이스는 미리 로드되고 메모리 상에 lock 되어 있으며, 가능하면 
      거대 페이지(huge page)를 사용한다.
  --enable-account-queries arg (=0)
    여러가지 메타데이터를 사용하여 계정을 찾는 질의를 사용할 수 있게 함.
  --max-nonprivileged-inline-action-size arg (=4096)
    특권 없는 계정의 인라인 액션의 최대 크기(bytes 단위)를 설정.
  --transaction-retry-max-storage-size-gb arg 
    트랜잭션 재시도(Transaction Retry) 기능에게 허용된 최대 할당 크기(GiB 단위).
    0 이상의 값이면 활성화 된다.
  --transaction-retry-interval-sec arg (=20)
    incoming 트랜잭션이 블록 내부에 보이지 않는다면 얼마나 자주(초 단위) 네트워크로 
    재전송(resend) 할지를 설정.
  --transaction-retry-max-expiration-sec arg (=90)
    허용된 트랜잭션 재시도 만료 시간(초 단위). 이 시간동안 트랜잭션을 재시도한다.
  --transaction-finality-status-max-storage-size-gb arg
    트랜잭션 최종성 상태(Transaction Finality Status) 기능에게 허용된 최대 할당 크기(GiB 단위).
    0 이상의 값이면 활성화된다.                                        
  --transaction-finality-status-success-duration-sec arg (=180)
    성공한 트랜잭션이 처음 확인된 시점부터 최종성 상태(Finality Status)가 유효한 기간(초 단위).
  --transaction-finality-status-failure-duration-sec arg (=180)
    실패한 트랜잭션이 처음 확인된 시점부터 최종성 상태(Finality Status)가 유효한 기간(초 단위).
  --block-log-retain-blocks arg
    블록 로그를 주기적으로 삭제하고 설정된 수 만큼의 최신 블록만 저장.
```

## 의존성

없음
