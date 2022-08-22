# Producer Plugin

## 개요

`producer_plugin` 을 설정하면 nodeos 가 블록 생산자(Block Producer, BP)로서 블록을 생성하는데 필요한 기능을 사용할 수 있습니다.

{% hint style="info" %}
블록 생성을 하려면 플러그인 설정 외에도 추가 설정이 더 필요합니다. 상세한 내용은 [블록 생산자(BP) 노드 설정](../setup-producer-node.md) 단원을 참조합니다.
{% endhint %}

## 사용법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::producer_plugin [options]

# nodeos startup params
nodeos ... -- plugin eosio::producer_plugin [options]
```

## Producer Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::producer_plugin 설정 옵션

  -e [ --enable-stale-production ]      
    블록을 생성한다.(체인이 최신상태가 아니라도)
  -x [ --pause-on-startup ]             
    블록 생성이 일시 정지 상태가 되었된 위치부터 이 노드를 시작한다.
  --max-transaction-time arg (=30)      
    푸시된 트랜잭션의 코드가 무효한 것으로 간주되기 전에 실행할 수 있는 최대 시간(ms).
  --max-irreversible-block-age arg (=-1)
    이 노드가 블록을 생성할 체인에 대해 DPOS Irreversible Block 의 최대 사용 기간(초)을 제한한다.
    (음수 값을 사용하면 무제한)
  -p [ --producer-name ] arg
    이 노드가 제어하는 블록 생성자 ID (e.g. inita; 여러 번 지정할 수 있음)
  --private-key arg
   (DEPRECATED) - signature-provider 를 대신 사용한다.
   [public key, WIF private key]의 튜플
  --signature-provider arg (=EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3)
    <public-key>=<provider-spec> 형태로 표현되는 Key=Value 쌍
    <public-key>    문자열 형태의 유효한 Antelope public key 
    <provider-spec> <provider-type>:<data> 형태의 문자열
    <provider-type> "KEY" 또는 "KEOSD"
    KEY:<data>      제공된 public key에 매핑되는, 문자열 형태의 유효한 Antelope private key.
    KEOSD:<data>    사용할 수 있는 KEOSD 가 있는 URL. 지갑은 unlocked 되어 있어야 한다.
  --keosd-provider-timeout arg (=5)
    서명하기 위해 keosd 공급자에게 블록을 보내는 데 허용되는 최대 시간(ms).
  --greylist-account arg
    확장된 CPU/NET 가상 리소스에 액세스할 수 없는 계정.
  --greylist-limit arg (=1000)
    사용률이 낮은 동안 CPU/NET 가상 리소스를 확장할 수 있는 배수(multiple)를 제한.
    (값은 1 ~ 1000 사이이며, 1000 을 적용하면 제한이 걸리지 않음)
  --produce-time-offset-us arg (=0)
    non last producing time 의 오프셋(ms). 유효한 범위: 0 .. block_time_interval
  --last-block-time-offset-us arg (=0)
    last producing time 의 오프셋(ms). 유효한 범위: 0 .. block_time_interval
  --cpu-effort-percent arg (=80)
    블록 생성에 사용되는 cpu 블록 생성 타임의 백분율. 숫자 자체가 백분율을 의미. 예를 들어 80 이면 80%.
  --last-block-cpu-effort-percent arg (=80)
    마지막 블록 생성에 사용되는 cpu 블록 생성 타임의 백분율. 숫자 자체가 백분율을 의미. 
    예를 들어 80 이면 80%.
  --max-block-cpu-usage-threshold-us (=5000)
    Block full 로 간주되는 CPU 블록 생성 임계값. max-block-cpu-usage 임계값 내에 있으면 
    즉시 블록을 생성할 수 있다.
  --max-block-net-usage-threshold-bytes (=1024)
    Block full 로 간주되는 NET 블록 생성 임계값. max-block-net-usage 임계값 내에 있으면 
    즉시 블록을 생성할 수 있다.
  --max-scheduled-transaction-time-per-block-ms arg (=100)
    정상적인 트랜잭션 프로세싱으로 돌아가기 전에 모든 블록에서 예약된 트랜잭션을 폐기하는 데 소요되는 
    최대 wall-clock time(ms).
  --subjective-account-max-failures arg (=3)
    블록당 주어진 계정에서 허용된 실패의 최대치를 설정.
  --subjective-account-decay-time-minutes arg (=1440)
    계정에 full subjective cpu 를 반환하는데 걸리는 시간을 설정.
  --incoming-defer-ratio arg (=1)
    수신되는 트랜잭션과 지연된 트랜잭션 둘 다 실행하기 위해 Queueing 되었을 때 두 트랜잭션의 간의 비율.
  --incoming-transaction-queue-size-mb arg (=1024)
    수신되는 트랜잭션 큐(Queue) 의 최대 크기(MiB). 이 크기를 넘어가면 리소스가 고갈되어 임의로 
    트랜잭션을 drop 시킨다.
  --disable-api-persisted-trx
    API 트랜잭션 재적용(re-apply)을 사용하지 않음.
  --disable-subjective-billing arg (=1)
    API/P2P 트랜잭션의 주관적 CPU 청구를 사용하지 않음.
  --disable-subjective-account-billing arg
    주관적 CPU 청구에서 제외되는 계정.
  --disable-subjective-p2p-billing arg (=1)
    P2P 트랜잭션에서 주관적 CPU 청구를 사용하지 않음.
  --disable-subjective-api-billing arg (=1)
    API 트랜잭션에서 주관적 CPU 청구를 사용하지 않음.
  --producer-threads arg (=2)
    BP 스레드풀에서 동작하는 worker 스레드의 수.
  --snapshots-dir arg (="snapshots")
    스냅샷 디렉토리 위치. (절대경로 또는 data 디렉토리의 상대경로)
```

## 의존성

* chain\_plugin

### 의존성 로딩 예제

다음과 같이 `chain_plugin` 을 사용하도록 설정되어 있어야 합니다.

```
# config.ini
plugin = eosio::chain_plugin [operations] [options]`

# command-line
nodeos ... --plugin eosio::chain_plugin [operations] [options]`
```

[블록 생성에 대한 상세](../block-producing-explained.md) 참조

## 트랜잭션 우선순위

지연된 트랜잭션이 `producer_plugin` 의 대기열에 있을 경우, 트랜잭션 타입에 따라 높은 우선 순위를 줄 수 있습니다. 아래 옵션은 지연된 트랜잭션과 수신되는 트랜잭션 간의 비율을 설정합니다.

`--incoming-defer-ratio arg (=1)`

예를 들면 다음과 같습니다.

* 기본값인 '1'로 설정하면 `producer_plugin` 은 하나의 지연된 트랜잭션 당 수신된 트랜잭션 하나를 처리합니다.&#x20;
* arg 을 10으로 설정하면`producer_plugin` 은 하나의 지연된 트랜잭션 당 10개의 수신된 트랜잭션을 처리합니다.

'arg' 가 적당히 큰 숫자로 설정된 경우 플러그인은 트랜잭션의 대기열이 빌 때까지 항상 수신되는 트랜잭션을 먼저 처리합니다. 'arg'가 0 이면 `producer_plugin` 은 대기열에 있는의 지연된 트랜잭션을 먼저 처리합니다.
