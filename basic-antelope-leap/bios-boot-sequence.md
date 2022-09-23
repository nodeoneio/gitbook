---
description: Bios Boot Sequence
---

# 바이오스 부트 시퀀스

## 개요

바이오스 부트 시퀀스는 하나 이상의 노드로 구성되는 Antelope 기반의 프라이빗 로컬 블록체인 네트워크를 만드는 방법으로, 크게 다음과 같이 두 단계를 거치게 됩니다.

1. 제네시스 BP 노드를 설치/구성하고 시작.
2. BP 를 추가하여 다수의 노드에서 일정에 맞춰 블록을 생성하도록 구성.

가장 먼저 제네시스 노드부터 만들어 보겠습니다.

{% hint style="info" %}
이 단원은 Ubuntu 20.04 LTS 기반으로 작성하였습니다.
{% endhint %}

## 제네시스 노드 구성하기

이 단계에서는 먼저 몇 가지 사전 준비 작업을 거친 후 다음과 같은 내용을 설정 할 것입니다.

* Antelope 노드 실행 환경을 설정.
* Antelope 제네시스 노드를 시작.
* 제네시스 노드와 연결하는 추가 Antelope BP 노드를 설정.

이 튜토리얼을 끝까지 진행하면 로컬에서 완전하게 동작하는 Antelope 기반 블록체인을 만들 수 있게 될 것입니다.

이 튜토리얼을 따라하면 스크립트가 무슨 일을 하는지 각 단계마다 상세한 설명을 볼 수 있습니다.

### 바이너리 인스톨

#### 사전 컴파일 된 Antelope Leap 바이너리 인스톨

아래 링크에 설명된 방법을 따라서 Leap 바이너리를 인스톨 합니다. 단, 현 단계에서는 인스톨만 하고 nodeos 를 실행 하지는 않습니다.

{% embed url="https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/~/changes/FJVBH0OqxreZ45Nlpwy2/basic-antelope-leap/install-leap-software/leap" %}

#### CDT 바이너리 인스톨

아래 링크 내용을 따라서 CDT 바이너리를 인스톨 합니다.

{% embed url="https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/~/changes/FJVBH0OqxreZ45Nlpwy2/cdt/install-cdt" %}

#### 개발용 지갑 생성

디폴트 지갑을 만들고 구성을 한 뒤 개발용 키 쌍를 만든 뒤 지갑으로 키를 가져옵니다. 이 단원에서는 편의상 앞으로 공개키를 `EOS_PUB_DEV_KEY` 라고 하고 개인키를 `EOS_PRIV_DEV_KEY` 라고 하겠습니다.

먼저 디폴트 지갑을 만들어 봅시다. 디폴트 지갑은 다음과 같이 만듭니다.

```
$ cleos wallet create --to-console
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5KKKpPwUQ5jEEYANZVeDtP3VUPchyCdj7icdPv5jgbGEyKSftLb"
```

`--to-console` 옵션을 사용하면 지갑의 패스워드가 콘솔 화면에 표시됩니다. 패스워드가 화면에 표시되기를 원하지 않는다면 `--to-file` 을 사용하여 파일로 출력할 수 있습니다. 지갑 관리 데몬인 keosd 를 따로 실행하지 않았더라도 위 명령을 실행하면 자동으로 keosd 가 실행됩니다.

이 단원에서 사용할 키는 다음과 같습니다.

* `EOS_PUB_DEV_KEY` : EOS6Pxs3oiKT7y6eP58qr6KzYSPA5hbe7XtDciNNFM8qrVwPWBszW
* `EOS_PRIV_DEV_KEY`: 5HtKjCM8k54K3ZQziNBaACCKgzGieWHFMzakiFkQXXzfGVeqczG

진행상의 편의를 위해 이 단원에서 사용하는 모든 키 쌍은 위 키를 사용할 것입니다. 이 키는 개발/테스트 외 프로덕션 용으로 사용해서는 안 됩니다. 만약 다른 키를 사용하고 싶다면 `cleos create key` 명령으로 새 키를 만들 수 있습니다.&#x20;

키를 디폴트 지갑으로 가져오려면 다음과 같이 `cleos wallet import` 명령을 실행하고 `private key:` 프롬프트에서 개인키를 붙여넣으면 됩니다.

{% code overflow="wrap" %}
```
$ cleos wallet import
private key: imported private key for: EOS6Pxs3oiKT7y6eP58qr6KzYSPA5hbe7XtDciNNFM8qrVwPWBszW
```
{% endcode %}

{% hint style="info" %}
지갑은 15분동안 사용하지 않으면 잠금 상태가 됩니다. 만약 지갑의 잠금 해제 시간을 늘리고 싶다면 keosd 의 환경 설정 파일(디폴트로 `~/eosio-wallet/config.ini`) 에서 `unlock-timeout` 값(초 단위)을 원하는 만큼 늘려주면 됩니다.&#x20;

변경하고 나서는 keosd 데몬의 PID 를 찾아서 `kill PID` 명령으로 데몬을 제거한 뒤, 적당한 지갑 관련 명령(ex: `cleos wallet list`) 을 실행하면 설정이 적용된 keosd 가 실행됩니다.
{% endhint %}

#### Leap 데이터 디렉토리 작성

다음과 같이 `/home/nodeos` 디렉토리를 만듭니다. 제네시스 노드를 시작할 때 이런저런 매개변수들을 사용하여 제네시스 노드가 블록체인 데이터베이스를 만들 때 사용할 로그 파일과 설정 파일을 이 디렉토리 내부에 저장할 것입니다.

```
mkdir -p /home/nodeos/genesis
cd /home/nodeos/genesis
```

{% hint style="info" %}
root 사용자가 아닌 경우 데이터 디렉토리 작성시 sudo 명령을 사용하여 데이터 디렉토리를 만든 뒤, 다음과 같이 현재 사용자의 소유로 변경해 줍니다.\
\
`sudo chown -R <username>:<username> /home/nodeos/`
{% endhint %}

#### **데이터 디렉토리에 제네시스 파일 작성**

데이터 디렉토리(`/home/nodeos/genesis`)에서 genesis.json 파일을 작성합니다.

```
nano genesis.json
```

다음의 jSON 파일을 복사하여 제네시스 파일에 붙여넣고 저장합니다.

```
{
  "initial_timestamp": "2022-08-29T09:00:00.000",
  "initial_key": "<EOS_PUB_DEV_KEY>",
  "initial_configuration": {
    "max_block_net_usage": 1048576,
    "target_block_net_usage_pct": 1000,
    "max_transaction_net_usage": 524288,
    "base_per_transaction_net_usage": 12,
    "net_usage_leeway": 500,
    "context_free_discount_net_usage_num": 20,
    "context_free_discount_net_usage_den": 100,
    "max_block_cpu_usage": 100000,
    "target_block_cpu_usage_pct": 500,
    "max_transaction_cpu_usage": 50000,
    "min_transaction_cpu_usage": 100,
    "max_transaction_lifetime": 3600,
    "deferred_trx_expiration_window": 600,
    "max_transaction_delay": 3888000,
    "max_inline_action_size": 4096,
    "max_inline_action_depth": 4,
    "max_authority_depth": 6
  }
}
```

위 내용 중 `initial_key` 항목의 “`EOS_PUB_DEV_KEY`” 를 앞서 '개발용 지갑 생성' 파트에서 개발용 지갑에 넣었던 공개키로 바꿉니다. 그리고 파일을 저장하고 나갑니다.

#### 환경설정 파일 작성

노드의 실행 옵션은 명령줄이나 환경설정 파일에서 지정할 수 있습니다. 다음과 같이 데이터 디렉토리 안에 `config.ini` 파일을 만듭니다.

```
nano config.ini
```

그리고 아래 내용을 붙여넣습니다.&#x20;

```
wasm-runtime = eos-vm
abi-serializer-max-time-ms = 15
chain-state-db-size-mb = 65536
contracts-console = true
http-server-address = 0.0.0.0:8888
p2p-listen-endpoint = 0.0.0.0:9876
state-history-endpoint = 0.0.0.0:8080
verbose-http-errors = true
agent-name = "Nodeos Local"
net-threads = 2
max-transaction-time = 100
enable-stale-production = true
resource-monitor-not-shutdown-on-threshold-exceeded=true

http-validate-host = false
http-threads = 6
access-control-allow-origin = *
access-control-allow-headers = Origin, X-Requested-With, Content-Type, Accept
http-max-response-time-ms = 100

#p2p-peer-address = <p2p address> :9876
signature-provider=<EOS_PUB_DEV_KEY>=KEY:<EOS_PRIV_DEV_KEY>
producer-name = eosio

plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
```

`signature-provider` 옵션의 `EOS_PUB_DEV_KEY` 와 `EOS_PRIV_DEV_KEY` 를 각각의 키로 변경합니다.

그리고 `chain-state-db-size-mb` 에 로컬 머신의 메모리 용량을 입력합니다. 메모리는 다음과 같이 계산할 수 있습니다,

{% code overflow="wrap" %}
```
awk '/MemTotal/ {printf( "%.2f\n", $2 / 1024 )}' /proc/meminfo |  awk '{print int($0)}'
```
{% endcode %}

로컬 테스트넷이라면 16GB(16384) 이하로도 충분합니다. 다만 입력할 메모리 용량이 전체 물리 메모리양을 넘어가지 않도록 합니다.&#x20;

{% hint style="info" %}
위 제네시스 노드 설정 내용을 간략히 정리하면 다음과 같습니다.

* 블록을 생성합니다.
* 0.0.0.0:8888 에서 HTTP 요청을 받습니다.
* 0.0.0.0:9876 에서 p2p 연결 요청을 받습니다.
* 0.0.0.0:8080 에서 블록 이력과 관련된 요청을 받습니다.
* 아직은 단독 노드이기 때문에 `p2p-peer-address` 에서 p2p 연결은 하지 않습니다. 나중에 BP 노드를 추가하게 되면 그 때 p2p 노드 정보를 추가할 것입니다.
* 스마트 컨트랙트의 출력을 콘솔에 표시하는 `--contracts-console` 매개 변수가 있는데, 여기서 표시하는 정보는 문제가 발생했을 때 해결의 실마리가 될 수 있습니다.
{% endhint %}

이제 작성한 내용을 저장하고 빠져나옵니다.

#### 노드를 시작/중지하는 스크립트 작성

노드를 실행하기 전에 노드 시작과 중지를 간편하게 해 주는 두 개의 스크립트를 다음과 같이 작성합니다.

* **start.sh**

{% code overflow="wrap" %}
```
#!/bin/bash

# option
# 스냅샷에서 시작할 경우: -s [snapshot path]
# Genesis.json 에서 시작할 경우: -g [genesis.json path]

NODEOSBINDIR="/usr/local/bin"
DATADIR="/home/nodeos/genesis"
SNAPSHOT=""
GENESIS=""

$DATADIR/stop.sh
printf "Starting Nodeos \n";

ulimit -n 65535
ulimit -s 64000

while getopts g:s: flag
do
    case "${flag}" in
        s) SNAPSHOT="--snapshot ${OPTARG}";;
        g) GENESIS="--genesis-json ${OPTARG}";;
    esac
done

$NODEOSBINDIR/nodeos $GENESIS $SNAPSHOT --data-dir $DATADIR --config-dir $DATADIR 2>> $DATADIR/stderr.txt &  echo $! > $DATADIR/nodeos.pid
```
{% endcode %}

* **stop.sh**

```
#!/bin/bash

DIR="/home/nodeos/genesis"

if [ -f $DIR"/nodeos.pid" ]; then
  pid=`cat $DIR"/nodeos.pid"`
  kill $pid

  printf "Stopping Nodeos [$pid]"

  while true; do
      [ ! -d "/proc/$pid/fd" ] && break
      printf "."
      sleep 1
  done
  printf "\n"
  rm -r $DIR"/nodeos.pid"

  DATE=$(date -d "now" +'%Y_%m_%d-%H_%M')
  if [ ! -d $DIR/logs ]; then
      mkdir $DIR/logs
  fi
  tar -pcvzf $DIR/logs/stderr-$DATE.txt.tar.gz stderr.txt

  printf "\rNodeos Stopped.    \n"
fi
```

두 스크립트를 작성한 후 실행 권한 (ex: `chmod +x start.sh`) 를 부여합니다.&#x20;

#### 노드 실행 방법

다음과 같이 제네시스 노드를 실행합니다.

```
./start.sh -g genesis.json
```

노드가 정상적으로 실행중이라면 같은 디렉토리 안에 `stderr.txt` 파일이 생성될 것이며, 여기에 nodeos 의 콘솔 로그가 출력될 것입니다. `tail -f stderr.txt` 명령으로 로그가 기록되는 상황을 실시간으로 확인할 수 있습니다.

#### 노드 중지 방법

노드를 중지하려면 다음과 같이 실행합니다.&#x20;

```
./stop.sh
```

만약 nodeos 데몬을 강제 종료하면 블록체인 상태 데이터베이스가 망가질 수 있기 때문에 가능한 스크립트로 안전하게 종료 하는 것이 좋습니다.

#### 노드를 재시작 하는 방법

앞서 nodeos 프로세스를 시작할 때 `start.sh` 에서 `-g` 옵션을 추가하여 제네시스 파일을 지정하였습니다. 다시 시작할 때는 이 옵션을 빼고 다음과 같이 `start.sh` 만 실행하면 됩니다.

```
./start.sh
```

#### nodeos 재시작 시 오류 해결 방법

* `"perhaps we need to replay"`: 이 메시지는 nodeos 를 `-hard-replay-blockchain` 옵션 없이 시작했을 때 나타날 수 있습니다. 이 옵션은 제네시스 블록부터 모든 트랜잭션을 재생하는 옵션입니다. 이 오류를 없애고 싶다면 `-hard-replay-blockchain` 파라미터를 `config.ini` 환경설정 파일에 추가합니다.

{% hint style="info" %}
nodeos 를 재시작할 때 다음과 같은 옵션들을 사용할 수 있습니다.

\--truncate-at-block

\--delete-all-blocks

\--replay-blockchain

\--hard-replay-blockchain
{% endhint %}

#### 처음부터 노드를 재시작하기.

노드를 처음부터 다시 시작하려면 현재 노드의 블록 로그 파일과 상태 파일을 먼저 삭제해야 합니다.

먼저 다음과 같은 내용으로 `clean.sh` 라는 스크립트 파일을 만듭니다. 그리고 적절한 실행 권한을 줍니다.

```
#!/bin/bash
rm -rf /home/nodeos/genesis/blocks/*
rm -rf /home/nodeos/genesis/state/*
rm /home/nodeos/genesis/stderr.txt
touch /home/nodeos/genesis/stderr.txt
```

다음으로 `stop.sh` 스크립트를 실행하여 블록체인을 중지한 후 clean.sh 스크립트를 실행하여 현재 설정 파일 및 로그, 블록체인 데이터를 지웁니다. 이는 다음과 같은 순서로 진행하면 됩니다.

```
cd /home/nodeos/genesis
./stop.sh
./clean.sh
./start.sh -g genesis.json
```

#### stderr.txt 파일 확인하기

다음 명령으로 stderr.txt 파일에 기록되는 로그를 볼 수 있습니다.

```
cd /home/nodeos/genesis
tail -f stderr.txt
```

#### 핵심 시스템 계정

노드를 운영하려면 다음과 같은 시스템 계정들이 필요합니다.

```
eosio.bpay
eosio.msig
eosio.names
eosio.ram
eosio.ramfee
eosio.saving
eosio.stake
eosio.token
eosio.vpay
eosio.rex
```

#### 시스템 계정 만들기

시스템 계정에는 다음 키 쌍을를 사용할 것입니다.

```
Private key: 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
Public key: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
```

다음과 같이 입력하여 이 키를 디폴트 지갑에 넣습니다.

{% code overflow="wrap" %}
```
cleos wallet import --private-key 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
```
{% endcode %}

디폴트 지갑에 키가 성공적으로 들어가면 해당 키의 공개키가 콘솔상에 표시 됩니다.

{% code overflow="wrap" %}
```
private key: imported private key for: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
```
{% endcode %}

다른 키를 사용하려면 다음 명령어를 명령줄에 입력하여 키를 생성합니다.

```
cleos create key --to-console
```

이제 지갑에 넣은 키를 가지고 계정을 만들 수 있습니다. 아래 명령을 실행하여 각각의 시스템 계정을 생성합니다. 이 단원에서는 계정의 `owner` 키와 `active` 키를 동일한 키 쌍으로 지정할 것이기 때문에 계정 생성 명령에 키를 한 번만 지정하면 됩니다. 하지만 대부분의 일반적인 계정에서는 보안상 `owner` 키와 `active` 키를 분리하는 것이 더 좋습니다.

이 스크립트에서는 모든 eosio.\* 계정이 모두 같은 키를 사용하도록 할 것이지만 원한다면 다른 키를 만들어 사용해도 됩니다.

{% code overflow="wrap" %}
```
cleos create account eosio eosio.bpay EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.msig EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.names EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.ram EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.ramfee EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.saving EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.stake EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.token EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.vpay EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
cleos create account eosio eosio.rex EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
```
{% endcode %}

### 시스템 컨트랙트 빌드 및 배포하기

Antelope 블록체인의 기본 시스템 스마트 컨트랙트를 인스톨 하려면 먼저 시스템 컨트랙트 소스를 다운로드 받고 컴파일 한 후 몇 가지 프로토콜 기능을 활성화해야 합니다.

먼저 스마트 컨트랙트 빌드를 위한 도구를 설치합니다.

```
sudo apt install build-essential cmake
```

#### eosio.contracts 빌드

`eosio.contracts` 저장소를 클론하고 저장소에 포함된 빌드 스크립트로 소스를 빌드합니다.&#x20;

{% code overflow="wrap" %}
```
cd /home/nodeos/genesis
git clone https://github.com/eosnetworkfoundation/eos-system-contracts.git
cd ./eos-system-contracts
./build.sh # 빌드 중 eosio.cdt 설치 디렉토리를 입력해야 할 수도 있는데, /usr/opt/cdt/<version> 아래에 설치되어 있다.
cd ./build/contracts/
```
{% endcode %}

`build.sh` 파일로 빌드하는 도중 경고를 볼 수 있는데, 이는 리카르디안 컨트랙트가 없다는 경고로, 무시해도 문제 없습니다.

#### PREACTIVATE\_FEATURE 을 활성화

EOSIO v1.8 및 이후 버전에서 도입된 모든 프로토콜 기능을 사용하려면 특수한 프로토콜 기능(`PREACTIVATE_FEATURE`)을 먼저 활성화한 뒤 시스템 컨트랙트를 배포해야 합니다.

`PREACTIVATE_FEATURE` 을 활성화 하려면 다음 명령을 입력합니다.

{% code overflow="wrap" %}
```
curl --request POST \\
    --url <http://127.0.0.1:8888/v1/producer/schedule_protocol_feature_activations> \\
    -d '{"protocol_features_to_activate": ["0ec7e080177b2c02b278d5088611686b49d739925a92d9bfcacd7fc6b74053bd"]}'
```
{% endcode %}

#### eosio.boot 컨트랙트 설정

시스템 컨트랙트는 토큰 기반의 행동에 대한 모든 액션을 제공합니다. 시스템 컨트랙트를 배포하기 전 까지는 경제적 요건들을 고려할 필요 없이 액션을 수행할 수 있습니다.&#x20;

그러나 일단 시스템 컨트랙트가 활성화되면 액션을 실행할 때 경제적 요소들을 고려해야 합니다. 즉, 시스템 리소스(CPU, 네트워크, 메모리)에 대한 비용을 지불해야 하며 마찬가지로 계정을 새로 만들 때 비용을 지불해야 합니다. 시스템 컨트랙트를 통해 토큰을 스테이크 및 언스테이크 할 수 있고, 자원을 구입해야 하며, 잠재적 BP를 등록하고 나중에 투표할 수 있으며, BP 보상을 청구할 수 있고, 이런저런 권한과 제약을 설정할 수 있습니다.

이제 `eosio.boot` 컨트랙트를 설치하면 Antelope 기반 블록체인에서 권장되는 일련의 프로토콜 기능을 사용할 수 있게 됩니다.

{% code overflow="wrap" %}
```
//cleos set contract eosio.boot <eosio.boot.wasm 파일 경로>
cd /home/nodeos/genesis/eos-system-contracts/build/contracts/eosio.boot
cleos set contract eosio.boot .

Reading WAST/WASM from /home/nodeos/genesis/eos-system-contracts/build/contracts/eosio.boot/eosio.boot.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 2150ed87e4564cd3fe98ccdea841dc9ff67351f9315b6384084e8572a35887cc  39968 bytes  4395 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001be023060027f7e0060067f7e7e7f7f...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"buyrambytes","base":"","fields":[{"name":"p...
```
{% endcode %}

#### 다른 프로토콜 기능 활성화하기

이제 `eosio.boot` 와 `PREACTIVATE_FEATURE` 를 활성화하여 다른 프로토콜 기능과 시스템 스마트 컨트랙트를 사용할 수 있게 되었습니다.

어떤 프로토콜 기능을 사용하려면 `activate` 액션에 원하는 프로토콜 기능의 해시 다이제스트를 전달하여 활성화 시킵니다. Antelope 블록체인의 프로토콜 기능 다이제스트는 다음 명령으로 확인할 수 있습니다.

{% code overflow="wrap" %}
```
curl -X POST http://127.0.0.1:8888/v1/producer/get_supported_protocol_features | jq .
```
{% endcode %}

아래 예제는 Leap 3.1.0 에서 사용 가능한 프로토콜 기능을 활성화시키는 명령입니다. 무엇을 사용할 것인가는 어디까지나 선택사항이며 이러한 프로토콜 기능 없이도 블록체인을 사용할 수는 있지만, 가능하면 Antelope 기반 블록체인에서는 전부 활성화하여 사용하는 것이 좋습니다.

{% code overflow="wrap" %}
```
# ONLY_LINK_TO_EXISTING_PERMISSION
cleos push action eosio activate '["1a99a59d87e06e09ec5b028a9cbb7749b4a5ad8819004365d02dc4379a8b7241"]' -p eosio

# FORWARD_SETCODE
cleos push action eosio activate '["2652f5f96006294109b3dd0bbde63693f55324af452b799ee137a81a905eed25"]' -p eosio

# ACTION_RETURN_VALUE
cleos push action eosio activate '["c3a6138c5061cf291310887c0b5c71fcaffeab90d5deb50d3b9e687cead45071"]' -p eosio

# BLOCKCHAIN_PARAMETERS
cleos push action eosio activate '["5443fcf88330c586bc0e5f3dee10e7f63c76c00249c87fe4fbf7f38c082006b4"]' -p eosio

# GET_SENDER
cleos push action eosio activate '["f0af56d2c5a48d60a4a5b5c903edfb7db3a736a94ed589d0b797df33ff9d3e1d"]' -p eosio

# ONLY_BILL_FIRST_AUTHORIZER
cleos push action eosio activate '["8ba52fe7a3956c5cd3a656a3174b931d3bb2abb45578befc59f283ecd816a405"]' -p eosio

# RESTRICT_ACTION_TO_SELF
cleos push action eosio activate '["ad9e3d8f650687709fd68f4b90b41f7d825a365b02c23a636cef88ac2ac00c43"]' -p eosio

# DISALLOW_EMPTY_PRODUCER_SCHEDULE
cleos push action eosio activate '["68dcaa34c0517d19666e6b33add67351d8c5f69e999ca1e37931bc410a297428"]' -p eosio

 # FIX_LINKAUTH_RESTRICTION
cleos push action eosio activate '["e0fb64b1085cc5538970158d05a009c24e276fb94e1a0bf6a528b48fbc4ff526"]' -p eosio

 # REPLACE_DEFERRED
cleos push action eosio activate '["ef43112c6543b88db2283a2e077278c315ae2c84719a8b25f25cc88565fbea99"]' -p eosio

# NO_DUPLICATE_DEFERRED_ID
cleos push action eosio activate '["4a90c00d55454dc5b059055ca213579c6ea856967712a56017487886a4d4cc0f"]' -p eosio

# RAM_RESTRICTIONS
cleos push action eosio activate '["4e7bf348da00a945489b2a681749eb56f5de00b900014e137ddae39f48f69d67"]' -p eosio

# WEBAUTHN_KEY
cleos push action eosio activate '["4fca8bd82bbd181e714e283f83e1b45d95ca5af40fb89ad3977b653c448f78c2"]' -p eosio

# WTMSIG_BLOCK_SIGNATURES
cleos push action eosio activate '["299dcb6af692324b899b39f16d5a530a33062804e41f09dc97e9f156b4476707"]' -p eosio

# CONFIGURABLE_WASM_LIMITS2
cleos push action eosio activate '["d528b9f6e9693f45ed277af93474fd473ce7d831dae2180cca35d907bd10cb40"]' -p eosio

# GET_CODE_HASH
cleos push action eosio activate '["bcd2a26394b36614fd4894241d3c451ab0f6fd110958c3423073621a70826e99"]' -p eosio

# CRYPTO_PRIMITIVES
cleos push action eosio activate '["6bcb40a24e49c26d0a60513b6aeb8551d264e4717f306b81a37a5afb3b47cedc"]' -p eosio

# GET_BLOCK_NUM
cleos push action eosio activate '["35c2186cc36f7bb4aeaf4487b36e57039ccf45a9136aa856a5d569ecca55ef2b"]' -p eosio
```
{% endcode %}

만약 현재 활성화된 프로토콜 기능 목록을 확인하고 싶다면 다음과 같이 입력합니다.

```
curl http://127.0.0.1:8888/v1/chain/get_activated_protocol_features | jq .
```

#### eosio.token 컨트랙트 배포

이제 `eosio.token` 컨트랙트를 배포합니다. 이 컨트랙트는 토큰을 발행하거나 정보를 얻거나 이체할 수 있도록 합니다. `eosio.token` 을 배포하려면 다음과 같이 합니다.

{% code overflow="wrap" %}
```
//cleos set contract eosio.token <eosio.token.wasm 파일 경로>
cd /home/nodeos/genesis/eos-system-contracts/build/contracts/eosio.token
cleos set contract eosio.token .

Reading WAST/WASM from /home/nodeos/genesis/eos-system-contracts/build/contracts/eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 17fa4e06ed0b2f52cadae2cd61dee8fb3d89d3e46d5b133333816a04d23ba991  8024 bytes  974 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1560037f7e7f0060057f7e...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```
{% endcode %}

#### eosio.msig 컨트랙트 배포

`eosio.msig` 는 퍼미션 레벨과 다중서명(multisig) 프로세스를 사용할 수 있게 하고 또한 이들을 쉽게 정의하고 관리 할 수 있게 합니다. leap 3.1 에서 이 기능을 사용하려면 먼저 `CRYPTO_PRIMITIVE` 프로토콜 기능을 활성화해야 합니다.

{% code overflow="wrap" %}
```
# CRYPTO_PRIMITIVES
cleos push action eosio activate '["6bcb40a24e49c26d0a60513b6aeb8551d264e4717f306b81a37a5afb3b47cedc"]' -p eosio
```
{% endcode %}

이제 다음과 같이 `eosio.msig` 컨트랙트를 배포합니다.

{% code overflow="wrap" %}
```
//cleos set contract eosio.msig <eosio.msig.wasm 파일 경로>
cd /home/nodeos/genesis/eos-system-contracts/build/contracts/eosio.msig
cleos set contract eosio.msig .

Reading WAST/WASM from /users/documents/eos/build/contracts/eosio.msig/eosio.msig.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 007507ad01de884377009d7dcf409bc41634e38da2feb6a117ceced8554a75bc  8840 bytes  925 us
#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d010000000198011760017f0060047f7e7e7...
#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":{"types":[{"new_type_name":"account_name","type":"name"}],"structs":[{...
```
{% endcode %}

#### SYS 화폐를 만들고 할당

이제 새로운 토큰을 만들 준비가 되었습니다. 여기서는 SYS 화폐를 만들고 토큰을 최대 100억 개 까지 발행할 수 있도록 설정해 보겠습니다. 그리고 일단 먼저 10억개를 발행해 보도록 하겠습니다. 원한다면 SYS 는 다른 화폐명으로 바꿔도 상관없습니다.

첫 단계로 `eosio.token` 계정을 사용하여 인증하게 되는 `eosio.token` 컨트랙트의 `create` 액션을 사용하여 100억개의 SYS 토큰을 생성합니다. 이는 토큰의 최대 공급량을 설정하는 것으로 실제로 토큰을 발행하지는 않습니다. 발행되지 않은 토큰은 일종의 적립금인 상태라고 생각하면 됩니다.

{% code overflow="wrap" %}
```
cleos push action eosio.token create '[ "eosio", "10000000000.0000 SYS" ]' -p eosio.token@active

executed transaction: 0440461e0d8816b4a8fd9d47c1a6a53536d3c7af54abf53eace884f008429697  120 bytes  326 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 SYS"}
```
{% endcode %}

두 번째로, `eosio.token` 컨트랙트의 `issue` 액션을 사용하여 10억 개의 SYS 토큰을 적립된 상태로부터 발행하도록 합니다. 발행한 순간부터 토큰들은 `eosio` 계정에 귀속됩니다. `eosio` 계정이 아직 발행되지 않은, 적립된 토큰을 가지고 있기 때문에 액션을 수행하려면 이 계정으로 인증해야 합니다.

{% code overflow="wrap" %}
```
cleos push action eosio.token issue '[ "eosio", "1000000000.0000 SYS", "memo" ]' -p eosio@active

executed transaction: a53961a566c1faa95531efb422cd952611b17d728edac833c9a55582425f98ed  128 bytes  432 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 SYS","memo":"memo"}
```
{% endcode %}

{% hint style="info" %}
경제적 관점에서 보면, 토큰을 발행함 으로서 유동성을 공급하면, 즉 토큰을 적립금에서 유통되는 상태로 이동시키면, 그로 인해 인플레이션이 발생하게 됩니다. 토큰 발행은 인플레이션을 발생시키는 하나의 방법입니다.
{% endhint %}

#### eosio.system 컨트랙트 배포

마지막으로 다음과 같이 `eosio` 계정에 `eosio.system` 컨트랙트를 배포합니다.

```
//cleos set contract eosio <eosio.system.wasm 파일 경로>
cd /home/nodeos/genesis/eos-system-contracts/build/contracts/eosio.system
cleos set contract eosio .
```

이것으로 단일 제네시스 노드로 동작하는 블록체인 설정을 완료하였습니다.

## 단일 제네시스 BP에서 다중 BP로 전환하기

이제 단일 BP(제네시스 노드)에서 다중 BP로 전환하는 방법을 알아보겠습니다. 지금까지는 기본으로 제공된 eosio 계정만 권한이 있으면 블록에 서명할 수 있었습니다. 이번 단원의 목표는 2/3+1의 BP 들이 무엇이 최종 블록인지에 동의하는 규칙에 따라 운영되는, 선출된 다수의 블록 프로듀서(BP)들이 모여 블록체인을 관리하는 것이 목표입니다.

BP 는 투표에 의해 선정되며 그 결과에 따라 BP 목록이 변경될 수 있습니다. Antelope 의 거버넌스 규칙 하에서는 특정 BP에게 직접 권한을 부여하기 보다는 기본으로 제공되는 `eosio.prods` 라는 특별한 계정으로 연결됩니다. 이 계정은 선출된 프로듀서 그룹을 나타냅니다. 사실상 BP들의 그룹이나 마찬가지인 `eosio.prods` 계정은 `eosio.msig` 계약에 의해 정의된 권한을 운영하고 사용합니다.

`eosio.system` 컨트랙트를 배포한 후에 `eosio.msig` 를 특권 있는 계정으로 지정하면 `eosio` 계정 대신 이 계정으로 권한을 부여할 수 있습니다. 이렇게 하여 `eosio` 계정은 원래 가지고 있던 권한을 내려놓게 되고 `eosio.prods` 계정이 그 권한을 대신하게 됩니다.

### 특권 있는 계정으로 eosio.msig 지정

`eosio.msig` 을 특권 있는 계정으로 지정하려면 다음과 같이 합니다.

```
cleos push action eosio setpriv '["eosio.msig", 1]' -p eosio@active
```

### 시스템 계정 초기화

다음 명령은 system 계정을 코드 제로(초기화 시간에 필요함) 로 초기화하고 SYS 토큰의 정밀도(소수점 이하 자릿수)를 4로 지정합니다. 정밀도는 0\~18까지 지정할 수 있습니다.

```
cleos push action eosio init '["0", "4,SYS"]' -p eosio@active
```

### 토큰 스테이킹과 네트워크 확장하기

여기까지 잘 따라 오셨다면 다음과 같은 컨트랙트가 설치된 단일 호스트의, 단일 제네시스 노드 구성이 완료 되었을 것입니다.

* eosio.token
* eosio.msig
* eosio.system

`eosio` 와 `eosio.msig` 는 특권을 가진 계정이며 다른 `eosio.*` 계정들은 특권을 가진 계정이 아닙니다.

이제 스테이크를 하고 BP 네트워크를 확장 할 준비가 되었습니다.

### 스테이크 된 계정 만들기

스테이크 한다는 것은 Antelope 블록체인 시스템 내의 계정에 토큰을 할당하는 프로세스를 의미합니다. 스테이킹과 언스테이킹은 블록체인의 수명 전반에 걸쳐 진행되는 과정이지만, 바이오스 부트 시퀀스 과정에서 수행되는 초기 스테이킹은 조금 특별합니다.&#x20;

바이오스 부트 시퀀스 중에는 계정의 토큰이 스테이킹 됩니다. 하지만, BP가 선출되기 전까지 이 토큰은 사실상 동결 상태에 있게 됩니다. 즉, 바이오스 부트 시퀀스 동안 수행되는 초기 스테이킹의 목표는, 토큰을 계정에 할당하여 사용 가능한 상태로 만들고, 투표 프로세스를 시작하여 BP가 선출되고 블록체인이 "라이브" 상태로 운영되도록 하는 것입니다.

다음은 초기 스테이킹 프로세스에서의 권장 사항입니다.

1. 0.1 토큰(계정 토큰의 10%가 아닌 문자 그대로 0.1 개)이 RAM 용으로 준비됩니다. 기본적으로 cleos 는 계정 생성 시 8KB의 RAM 을 스테이크 하며 이는 계정 생성자가 지불합니다. 초기 스테이킹에서 `eosio` 계정이 스테이킹을 하는 계정 생성자가 됩니다. 초기 토큰 스테이킹 프로세스 동안 스테이킹된 토큰은 최소 투표 요구 사항이 충족될 때까지는 언스테이킹하여 유동 자산화 할 수 없습니다.
2. CPU의 경우 0.45 토큰이, 네트워크의 경우 0.45 토큰이 스테이크 됩니다.
3. 그 다음으로 사용 가능한 토큰 중 총 9개까지 유동성 토큰으로 보유됩니다.
4. 나머지 토큰은 50/50 으로 CPU 및 네트워크에 할당됩니다.

> 예시 1.\
> accountnum11 계정은 100 SYS 를 가집니다. 이 계정은 0.1000 SYS 를 RAM 에, 45.4500 SYS 를 CPU 에, 45.4500 SYS 를 network 에 스테이킹 합니다. 그리고 9.0000 SYS 를 유동성 토큰으로 가집니다.
>
> 예시 2.\
> accountnum33 계정은 5 SYS 를 가집니다. 이 계정은 0.1000 SYS 를 RAM 에, 0.4500 SYS 를 CPU 에, 0.4500 SYS 를 network 에 스테이킹 합니다. 그리고 4.0000 SYS 를 유동성 토큰으로 가집니다.

학습 내용을 보다 사실적으로 구성하려면 10억 개의 토큰을 계정에 배포하고 파레토 분포를 사용할 수 있습니다. 파레토 분포는 80-20 규칙을 모델링 한 것으로, 예를 들어 토큰의 80%를 토큰 보유자의 20%가 가지게 됩니다. 이 예제에서는 분포를 생성하는 방법은 다루지 않을 것이며 스테이킹을 수행하는 명령에 초점을 맞추고 있습니다.

다음 단계를 따라 각 계정에서 토큰을 스테이크 합니다. 이 단계들은 각각의 계정마다 따로 수행해야 합니다.

{% hint style="info" %}
이 단원에서는 새로운 계정 생성 시 아래의 키 쌍을 사용할 것입니다. 실제 프로덕션 환경에서 사용해서는 안 되며 별도의 계정 키 값과 토큰을 만들어 사용해야 합니다.
{% endhint %}

{% code overflow="wrap" %}
```
Private key: 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
Public key: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt

cleos wallet import --private-key 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
imported private key for: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
```
{% endcode %}

새로운 계정 `bp.account.1` 을 다음과 같이 만듭니다.

{% code overflow="wrap" %}
```
cleos system newaccount eosio --transfer bp.account.1 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt --stake-net "100000000.0000 SYS" --stake-cpu "100000000.0000 SYS" --buy-ram-kbytes 8192

executed transaction: 07ec321e34d09e9becfdc3a15f4eacade2bfe14c56056040f609b1c5ed5754b5  336 bytes  2493 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"bp.account.1","owner":{"threshold":1,"keys":[{"key":"EOS8mUftJXepGzdQ2TaC...
#         eosio <= eosio::buyrambytes           {"payer":"eosio","receiver":"bp.account.1","bytes":8388608}
#         eosio <= eosio::delegatebw            {"from":"eosio","receiver":"bp.account.1","stake_net_quantity":"100000000.0000 SYS","stake_cpu_quant...
```
{% endcode %}

### 새로운 계정을 BP 로 등록하기

새로운 계정을 BP로 등록하려면 다음과 같이 합니다.

{% code overflow="wrap" %}
```
cleos system regproducer bp.account.1 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt https://www.nodeone.io 401

executed transaction: 43bebaf6a4302b08d639c1413faef7d30de130dac857d2f91e28c0b9153364cd  160 bytes  625 us
#         eosio <= eosio::regproducer           {"producer":"bp.account.1","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","u...
```
{% endcode %}

위 명령은 계정을 BP 후보로 만듭니다. 하지만 투표를 받아 선출되지 않는 이상 아직 블록을 생성하지는 않습니다.

### BP 목록 보기

다음 명령으로 BP 목록을 볼 수 있습니다. 현 시점에서는 BP로 등록된 계정만이 나타날 것입니다.

{% code overflow="wrap" %}
```
cleos system listproducers

Producer      Producer key                                           Url                                                         Scaled votes
accountnum11  EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt  <https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22> 0.0000
```
{% endcode %}

### 추가 BP 설정 및 시작

추가 BP를 설정하고 위에서 만든 bp.account.1 계정을 사용해 보겠습니다. 추가 BP를 설정하려면 다음과 같이 수행합니다.

```
cd /home/nodeos
mkdir bp.account.1
cd bp.account.1
copy /home/nodeos/genesis/genesis.json .
copy /home/nodeos/genesis/start.sh .
copy /home/nodeos/genesis/stop.sh .
copy /home/nodeos/genesis/clean.sh .
copy /home/nodeos/genesis/config.ini .
```

오류 없이 모든 과정이 제대로 수행된다면 다음과 같은 폴더가 보일 것입니다.

```
cd /home/nodeos/bp.account.1/
ls -al
drwxr-xr-x   8 owner  group   256 Dec  7 14:17 .
drwxr-xr-x   3 owner  group   960 Dec  5 10:00 ..
-rwxr-xr-x   1 owner  group   40  Dec  5 13:08 clean.sh
-rwxr-xr-x   1 owner  group   947 Dec  5 14:31 genesis.json
-rwxr-xr-x   1 owner  group   888 Dec  5 13:08 config.ini
-rwxr-xr-x   1 owner  group   901 Dec  6 15:44 start.sh
-rwxr-xr-x   1 owner  group   281 Dec  5 13:08 stop.sh
```

제네시스 노드에서 복사해 온 파일 중 다음 파일들의 내용을을 수정합니다.

**start.sh**

```
DATADIR="/home/nodeos/bp.account.1"
```

**stop.sh**

```
DIR="/home/nodeos/bp.account.1"
```

**clean.sh**

```
rm -rf /home/nodeos/bp.account.1/blocks/*
rm -rf /home/nodeos/bp.account.1/state/*
rm /home/nodeos/bp.account.1/stderr.txt
touch /home/nodeos/bp.account.1/stderr.txt
```

**config.ini**

{% code overflow="wrap" %}
```
p2p-peer-address = 10.117.126.249:9876
signature-provider=EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt=KEY:5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
producer-name = bp.account.1
```
{% endcode %}

#### 두 번째 BP 노드 시작하기

다음 명령으로 두 번째 BP노드를 시작합니다.

```
cd /home/nodeos/bp.account.1/
./start.sh -g genesis.json
tail -f stderr.txt
```

위 명령을 실행한 후 nodeos 가 생성하는 로그 스트림이 `stderr.txt` 에 출력되는 것을 콘솔에서 볼 수 있습니다. CTRL+C 키를 누르면 로그 스트림의 콘솔 출력을 중지합니다.

제네시스 노드와 마찬가지로 새 노드를 중지하려면 `stop.sh` 를 실행합니다. 노드를 다시 시작하려면 `start.sh` 스크립트를 `-g <파일명>` 옵션 없이 실행합니다.

모든 것을 지우고 처음부터 다시 시작하려면 다음 명령을 실행합니다.

```
cd /home/nodeos/bp.account.1/
./stop.sh
./clean.sh
./start.sh -g genesis.json
tail -f stderr.txt
```

### 여러 대의 BP 노드 만들기

이제 위의 과정을 반복하여 각각 자체 스테이크 계정, 자체 전용 디렉토리와 bp.accountXY 라고 명명된 계정, 자체 전용 스크립트 파일(`start.sh`, `stop.sh`, `clean.sh`)을 사용하여 원하는 만큼 BP를 생성할 수 있습니다.

또한 만들어낸 노드들이 p2p 로 서로 연결할 수 있어야 합니다. 다음의 `config.ini` 파일 옵션을 주의하여 설정합니다.

```
--producer-name <BP 계정 이름>
...
--http-server-address 0.0.0.0:8888 # API 요청을 받는 http 주소와 포트. 충돌하지 않도록 주의한다.
--p2p-listen-endpoint 0.0.0.0:9876 # p2p 연결을 위한 주소와 포트. 충돌하지 않도록 주의한다.
...
...
--p2p-peer-address localhost:9876    # 제네시스 노드 p2p
--p2p-peer-address localhost:9875    # bp.account.1 노드 p2p
--p2p-peer-address localhost:9874    # bp.account.2 노드 p2p
```

### 새로 생성한 BP에게 투표하기

이제 여러 노드들이 시작되었고 이들이 연결된 네트워크가 만들어 졌습니다. 그러나 제네시스 노드로 부터 블록을 전달받고는 있지만 아직 스스로 블록을 생성하고 있지는 않습니다.

#### 15% 요구사항

노드가 블록을 생성하려면 토큰 공급량의 총 15%를 스테이크한 후 준비가 된 모든 BP 들에게 투표해야 합니다. `bp.account.1` 계정에는 이미 이전에 할당된 토큰이 충분히 있습니다. BP 를 더 선출하려면 다음 명령을 실행합니다. 한 계정은 계정 이름으로 식별 가능한 최대 30명의 BP 에게 투표할 수 있습니다.

다음은 `bp.account.1` 이 `bp.account.1` \~ `bp.account.3` 에게 투표하는 예제입니다.

```
cleos system voteproducer prods bp.account.1 bp.account.1 bp.account.2 bp.account.3

```

투표가 완료되면 더 이상 eosio 노드는 블록을 생성하지 않고 투표를 받은 노드들이 블록을 생산하게 됩니다.

## eosio 계정 파기와 시스템 계정

BP 가 선출되고 최소한 15% 의 토큰이 투표를 위해 스테이킹 되어 있어야 한다 라는 최소한의 요구사항이 만족된 후, `eosio` 는 파기할 수 있습니다. 그러면 `eosio.msig` 만이 유일하게 특권을 가진 계정이 됩니다.

`eosio.*` 계정을 파기하려면 `eosio.*` 의 키 세트를을 null 로 설정하면 됩니다. 다음 명령을 사용하여 `eosio.*` 계정의 `owner` 키와 `active` 키를 삭제합니다.

{% code overflow="wrap" %}
```
cleos push action eosio updateauth '{"account": "eosio", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@owner
cleos push action eosio updateauth '{"account": "eosio", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@active
```
{% endcode %}

필요시 위에서 만든 시스템 계정들 역시 다음 명령을 사용하여 파기할 수 있습니다.

{% code overflow="wrap" %}
```
cleos push action eosio updateauth '{"account": "eosio.bpay", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.bpay@owner
cleos push action eosio updateauth '{"account": "eosio.bpay", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.bpay@active

cleos push action eosio updateauth '{"account": "eosio.msig", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.msig@owner
cleos push action eosio updateauth '{"account": "eosio.msig", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.msig@active

cleos push action eosio updateauth '{"account": "eosio.names", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.names@owner
cleos push action eosio updateauth '{"account": "eosio.names", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.names@active

cleos push action eosio updateauth '{"account": "eosio.ram", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.ram@owner
cleos push action eosio updateauth '{"account": "eosio.ram", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.ram@active

cleos push action eosio updateauth '{"account": "eosio.ramfee", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.ramfee@owner
cleos push action eosio updateauth '{"account": "eosio.ramfee", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.ramfee@active

cleos push action eosio updateauth '{"account": "eosio.saving", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.saving@owner
cleos push action eosio updateauth '{"account": "eosio.saving", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.saving@active

cleos push action eosio updateauth '{"account": "eosio.stake", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.stake@owner
cleos push action eosio updateauth '{"account": "eosio.stake", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.stake@active

cleos push action eosio updateauth '{"account": "eosio.token", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.token@owner
cleos push action eosio updateauth '{"account": "eosio.token", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.token@active

cleos push action eosio updateauth '{"account": "eosio.vpay", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.vpay@owner
cleos push action eosio updateauth '{"account": "eosio.vpay", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio", "permission": "active"}}]}}' -p eosio.vpay@active
```
{% endcode %}

## 모니터링하고 테스트하기

실행중인 nodeos (제네시스 노드와 추가한 BP 노드) 를 다음과 같이 모니터링 할 수 있습니다.

```
cd /home/nodeos/genesis/
tail -f stderr.txt

cd /home.nodeos/bp.account.1
tail -f stderr.txt
```

이제 여러 명령이나 계정 생성, 잔고 확인, 토큰 전송등을 테스트 할 수 있습니다.

새 계정을 만들려면 Create test account 튜토리얼을 확인합니다.

토큰을 발행하고 할당하거나고 계정간에 전송하려면 Deploy, Issue,and Transfer 섹션을 확인 합니다.

## 블록 익스플로러 연결하기

bloks.io 에는 로컬 테스트넷에 연결할 수 있는 기능이 있습니다. 이 기능을 사용하여 로컬 테스트넷을 쉽게 사용해 볼 수 있습니다.

[https://local.bloks.io/](https://local.bloks.io/)



## Reference

&#x20;__ [_BIOS Boot Sequence | EOSIO Developer Docs_](https://developers.eos.io/welcome/latest/tutorials/bios-boot-sequence/?query=bios%20boot%20sequence\&page=1#gatsby-focus-wrapper)
