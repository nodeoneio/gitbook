---
description: Bios Boot Sequence
---

# 바이오스 부트 시퀀스

## 개요

바이오스 부트 시퀀스는 하나 이상의 노드로 구성괴는 Mandel 기반의 프라이빗 로컬 블록체인 네트워크를 만드는 방법으로, 크게 다음과 같이 두 단계를 거치게 됩니다.

1. 제네시스 BP 노드를 설치/구성하고 시작.
2. BP 를 추가하여 다수의 노드에서 일정에 맞춰 블록을 생성하도록 구성.

가장 먼저 제네시스 노드부터 만들어 보겠습니다.

{% hint style="info" %}
이 단원은 Ubuntu 20.04 LTS 기반으로 작성하였습니다.
{% endhint %}

## 제네시스 노드 구성하기

이 단계에서는 먼저 몇 가지 사전 준비 작업을 거친 후 다음과 같은 내용을 설정 할 것입니다.

* Mandel 노드 실행 환경을 설정.
* Mandel 제네시스 노드를 시작.
* 제네시스 노드와에 연결하는 추가 Mandel 노드들을 설정.

이 튜토리얼을 끝까지 진행하면 로컬에서 완전하게 동작하는 Mandel 기반 블록체인을 만들 수 있게 될 것입니다.

{% hint style="info" %}
만약 모든 과정을 자동으로 실행 하고 싶다면 다음의 bios-boot-tutorial.py 파이썬 스크립트를 사용하여 준비 단계를 구성할 수 있습니다.&#x20;

https://github.com/EOSIO/eos/blob/release/2.1.x/tutorials/bios-boot-tutorial/bios-boot-tutorial.py

스크립트를 바로 쓸 수는 없고 별도의 외부 데이터값을 사용해야 합니다. BP 이름과 사용자 계정을 확인하려면 accounts.json 을 열어보면 됩니다. 자동으로 완전하게 동작하는 EOSIO 로컬 블록 체인을 만들고 싶다면 README.md 파일 내용을 따라서 bios-boot-tutorial.py 스크립트를 실행합니다.
{% endhint %}

이 튜토리얼을 따라하면 스크립트가 무슨 일을 하는지 각 단계마다 상세한 설명을 볼 수 있습니다.

### 바이너리 인스톨

#### 사전 컴파일 된 Mandel 바이너리 인스톨

아래 링크에 설명된 방법을 따라서 nodeos 바이너리를 인스톨 합니다. 단, 현 단계에서는 인스톨만 하고 nodeos 를 실행 하지는 않습니다.

[https://developers.eos.io/manuals/eos/latest/install/install-prebuilt-binaries](https://developers.eos.io/manuals/eos/latest/install/install-prebuilt-binaries)

#### CDT 바이너리

아래 링크 내용을 따라서 CDT 바이너리를 인스톨 합니다.

[https://developers.eos.io/manuals/eosio.cdt/latest/installation](https://developers.eos.io/manuals/eosio.cdt/latest/installation)

#### 개발용 지갑 생성

디폴트 지갑을 만들고 구성을 한 뒤 개발용 키 쌍를 만듭니다. 그리고 지갑으로 키를 가져옵니다. 이 단원에서는 편의상 앞으로 공개키를 EOS\_PUB\_DEV\_KEY 라고 하고 개인키를 EOS\_PRIV\_DEV\_KEY 라고 하겠습니다.

지갑을 만드는 방법과 지갑 안의 키들을 안전하게 보관하는 방법을 알아보려면 다음 튜토리얼을 확인합니다.

[https://developers.eos.io/welcome/latest/getting-started-guide/local-development-environment/development-wallet](https://developers.eos.io/welcome/latest/getting-started-guide/local-development-environment/development-wallet)

#### \~/biosboot/genesis 디렉토리 작성

다음과 같이 \~/biosboot/genesis 디렉토리를 만듭니다. 나중에 제네시스 노드를 시작할 때 특정한 파라미터 들을 사용할 것인데, 제네시스 노드가 블록체인 데이터베이스를 만들 때 사용할 로그 파일과 설정 파일이 이 디렉토리 내부에 저장될 것입니다.

```
cd ~
mkdir biosboot
cd biosboot
mkdir genesis
cd genesis
```

#### **\~/biosboot/ 디렉토리에 JSON 파일 작성**

빈 genesis.json 파일을 \~/biosboot/ 디렉토리 안에 작성합니다.

```
cd ~/biosboot
touch genesis.json
nano genesis.json
```

다음의 jSON 파일을 복사합니다.

```
{
  "initial_timestamp": "2018-12-05T08:55:11.000",
  "initial_key": "EOS_PUB_DEV_KEY",
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
  },
  "initial_chain_id": "0000000000000000000000000000000000000000000000000000000000000000"
}
```

genesis.json 파일에 위 내용을 붙여넣습니다. 위 내용 중 initial\_key 항목의 “EOS\_PUB\_DEV\_KEY” 를 앞서 '개발용 지갑 생성' 파트에서 개발용 지갑에 넣었던 공개키로 바꿉니다. 그리고 파일을 저장하고 나갑니다.

#### 제네시스 노드를 시작

다음 단계를 따라서 제네시스 노드를 만듭니다.

\~/biosboot/genesis/ 에 다음과 같이 genesis\_start.sh 쉘 스크립트 파일을 만듭니다.

```
cd ~/biosboot/genesis
touch genesis_start.sh
nano genesis_start.sh
```

다음의 스크립트를 복사하여 genesis\_start.sh 파일에 복사해 넣습니다.

```
#!/bin/bash
DATADIR="./blockchain"

if [ ! -d $DATADIR ]; then
  mkdir -p $DATADIR;
fi

nodeos \\
--genesis-json $DATADIR"/../../genesis.json" \\
--signature-provider EOS_PUB_DEV_KEY=KEY:EOS_PRIV_DEV_KEY \\
--plugin eosio::producer_plugin \\
--plugin eosio::producer_api_plugin \\
--plugin eosio::chain_plugin \\
--plugin eosio::chain_api_plugin \\
--plugin eosio::http_plugin \\
--plugin eosio::history_api_plugin \\
--plugin eosio::history_plugin \\
--data-dir $DATADIR"/data" \\
--blocks-dir $DATADIR"/blocks" \\
--config-dir $DATADIR"/config" \\
--producer-name eosio \\
--http-server-address 0.0.0.0:8888 \\
--p2p-listen-endpoint 0.0.0.0:9010 \\
--access-control-allow-origin=* \\
--contracts-console \\
--http-validate-host=false \\
--verbose-http-errors \\
--enable-stale-production \\
--p2p-peer-address [PEER_ADDR1:9010] \\
--p2p-peer-address [PEER_ADDR2:9010] \\
--p2p-peer-address [PEER_ADDR3:9010] \\
>> $DATADIR"/nodeos.log" 2>&1 & \\
echo $! > $DATADIR"/eosd.pid"
```

위 스크립트의 EOS\_PUB\_DEV\_KEY 와 EOS\_PRIV\_DEV\_KEY 를 1.2 단계에서 만든 공개키와 개인키로 바꿉니다.

또한 나중에 --p2p-peer-address 의 \[PEER\_ADDR1,2,3] 에 p2p 로 연결할 노드들의 주소를 기입할 것입니다.

이제 파일을 저장하고 나옵니다.

genesis\_start.sh 쉘 스크립트에 실행 권한을 주고 실행하면 nodeos 를 시작할 수 있습니다.

```
cd ~/biosboot/genesis/
chmod +x genesis_start.sh
./genesis_start.sh
```

{% hint style="info" %}
여기서 작성한 제네시스 노드의 특성은 다음과 같습니다.

* 블록을 생성합니다.
* 0.0.0.0:8888 에서 HTTP 요청을 받습니다.
* 0.0.0.0:9010 에서 p2p 연결 요청을 받습니다.
* PEER\_ADDR1:9010, PEER\_ADDR2:9010, PEER\_ADDR3:9010 에 주기적으로 p2p 연결을 시도합니다. 아직 이 노드들이 실행되고 있지는 않으므로 연결 시도가 실패하더라도 무시합니다.
* 스마트 컨트랙트의 출력을 콘솔에 표시하는 --contracts-console 매개 변수가 있는데, 여기서 표시하는 정보는 문제가 발생했을 때 해결의 실마리가 될 수 있습니다.
{% endhint %}

#### 제네시스 노드 중지 방법

nodeos 를 중지하려면 다음과 같이 설정합니다.

\~/biosboot/genesis/ 에 [stop.sh](http://stop.sh) 쉘 스크립트 파일을 만들고 다음 내용을 복사해 넣습니다.

```
#!/bin/bash
DATADIR="./blockchain/"

if [ -f $DATADIR"/eosd.pid" ]; then
pid=`cat $DATADIR"/eosd.pid"`
echo $pid
kill $pid
rm -r $DATADIR"/eosd.pid"
echo -ne "Stoping Node"
while true; do
[ ! -d "/proc/$pid/fd" ] && break
echo -ne "."
sleep 1
done
echo -ne "\\rNode Stopped. \\n"
fi
```

[stop.sh](http://stop.sh) 쉘 스크립트에 적절한 권한을 주고 실행합니다.

```
cd ~/biosboot/genesis/
chmod +x stop.sh
./stop.sh
```

#### 노드를 재시작 하는 방법

일단 nodeos 프로세스를 시작한 다음 중지하게 되면 앞서 만든 genesis\_start.sh 스크립트로 다시 시작할 수 없습니다. 일단 노드가 실행되고 블록을 생성하면 블록체인 데이터베이스가 초기화되고 채워지기 시작하는데, 이 상태가 되면 nodeos 를 시작할 때 start.sh 에서 --genesis-json 파라미터를 사용할 수 없게 되므로 노드를 재시작 할 스크립트를 별도로 만드는 것이 좋습니다. 앞서 설명한 것과 동일한 단계를 따라 제네시스 노드를 시작하고 아래 내용으로 start.sh  스크립트를 만든 뒤 실행 권한을 할당합니다. 나중에 프로세스를 중지한 후 다시 시작할 때는 이 파일을 사용하면 됩니다.

```
#!/bin/bash
DATADIR="./blockchain"

if [ ! -d $DATADIR ]; then
  mkdir -p $DATADIR;
fi

nodeos \\
--signature-provider EOS_PUB_DEV_KEY=KEY:EOS_PRIV_DEV_KEY \\
--plugin eosio::producer_plugin \\
--plugin eosio::producer_api_plugin \\
--plugin eosio::chain_plugin \\
--plugin eosio::chain_api_plugin \\
--plugin eosio::http_plugin \\
--plugin eosio::history_api_plugin \\
--plugin eosio::history_plugin \\
--data-dir $DATADIR"/data" \\
--blocks-dir $DATADIR"/blocks" \\
--config-dir $DATADIR"/config" \\
--producer-name eosio \\
--http-server-address 127.0.0.1:8888 \\
--p2p-listen-endpoint 127.0.0.1:9010 \\
--access-control-allow-origin='*' \\
--contracts-console \\
--http-validate-host=false \\
--verbose-http-errors \\
--enable-stale-production \\
--p2p-peer-address localhost:9011 \\
--p2p-peer-address localhost:9012 \\
--p2p-peer-address localhost:9013 \\
>> $DATADIR"/nodeos.log" 2>&1 & \\
echo $! > $DATADIR"/eosd.pid"
```

#### nodeos 재시작 시 오류 해결 방법

* `"perhaps we need to replay"`: 이 메시지는 nodeos 를 `-hard-replay` 파라미터 없이 시작했을 때 나타날 수 있습니다. 이 파라미터는 제네시스 노드부터 모든 트랜잭션을 재생하는 옵션입니다. 이 오류를 없애고 싶다면 `-hard-replay` 파라미터를 `hard_replay.sh` 쉘 스크립트에 추가합니다.

{% hint style="info" %}
nodeos 를 재시작할 때 사용할 수 있는 파라미터들은 다음과 같습니다.

\--truncate-at-block

\--delete-all-blocks

\--replay-blockchain

\--hard-replay-blockchain
{% endhint %}

예를 들어 다음은 `-hard-replay-blockchain` 파라미터를 사용한  `hard_replay.sh` 쉘 스크립트입니다.

```
#!/bin/bash
DATADIR="./blockchain"

if [ ! -d $DATADIR ]; then
  mkdir -p $DATADIR;
fi

nodeos \\
--signature-provider EOS_PUB_DEV_KEY=KEY:EOS_PRIV_DEV_KEY \\
--plugin eosio::producer_plugin \\
--plugin eosio::producer_api_plugin \\
--plugin eosio::chain_plugin \\
--plugin eosio::chain_api_plugin \\
--plugin eosio::http_plugin \\
--plugin eosio::history_api_plugin \\
--plugin eosio::history_plugin \\
--data-dir $DATADIR"/data" \\
--blocks-dir $DATADIR"/blocks" \\
--config-dir $DATADIR"/config" \\
--producer-name eosio \\
--http-server-address 127.0.0.1:8888 \\
--p2p-listen-endpoint 127.0.0.1:9010 \\
--access-control-allow-origin=* \\
--contracts-console \\
--http-validate-host=false \\
--verbose-http-errors \\
--enable-stale-production \\
--p2p-peer-address localhost:9011 \\
--p2p-peer-address localhost:9012 \\
--p2p-peer-address localhost:9013 \\
--hard-replay-blockchain \\
>> $DATADIR"/nodeos.log" 2>&1 & \\
echo $! > $DATADIR"/eosd.pid"
```

#### 처음부터 nodeos 를 재시작하기.

다음 내용으로 [clean.sh](http://clean.sh) 라는 스크립트 파일을 만듭니다. 그리고 적절한 실행 권한을 줍니다.

```
#!/bin/bash
rm -fr blockchain
ls -al
```

현재 설정 파일 및 로그, 블록체인 데이터를 지우고 싶다면 먼저 [stop.sh](http://stop.sh) 스크립트를 실행하여 블록체인을 중지한 후 [clean.sh](http://clean.sh) 를 실행합니다. 이는 다음과 같은 순서로 진행하면 됩니다.

```
cd ~/biosboot/genesis/
./stop.sh
./clean.sh
./genesis_start.sh
```

#### nodeos.log 파일 확인하기

다음 명령으로 nodeos.log 파일에 기록되는 로그를 볼 수 있습니다.

```
cd ~/biosboot/genesis/
tail -f ./blockchain/nodeos.log
```

#### 핵심 시스템 계정 생성

다음과 같이 노드 운영에 필요한 시스템 계정들이 있습니다.

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

아래 순서를 반복하여 각각의 시스템 계정을 생성합니다. 이 튜토리얼에서는 owner 계정의 키와 active 계정의 키에 동일한 키 쌍을 사용할 것이기 때문에 계정 생성 명령에 키를 한 번만 지정하면 됩니다. 하지만 보안상 대부분의 일반적인 계정에서는 공개키와 개인키를 분리하는 것이 더 좋습니다.

이 스크립트에서는 모든 eosio.\* 계정이 같은 키를 사용하도록 할 것이지만 원한다면 다른 키를 사용해도 됩니다.

다음 명령어를 명령줄에 입력하여 키를 생성합니다.

```
cleos create key --to-console
```

그러면 다음과 같은 결과를 볼 수 있습니다.

```
Private key: 5KAVVPzPZnbAx8dHz6UWVPFDVFtU1P5ncUzwHGQFuTxnEbdHJL4
Public key: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
```

이 키를 지갑에 넣으려면 다음과 같이 입력합니다.&#x20;

```
cleos wallet import --private-key
```

그리고 이전 단계에서 만든 개인 키를 입력합니다. 그러면 디폴트 지갑으로 키를 가져오게 되며 해당 키의 공개키가 콘솔상에 표시 됩니다.

```
private key: imported private key for: EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG
```

이제 앞서 만든 키를 가지고 계정을 만들 수 있습니다. 다음은 eosio.bpay 계정을 만드는 명령이며 같은 식으로 위에 나열한 모든 계정을 만듭니다.

```
cleos create account eosio eosio.bpay EOS84BLRbGbFahNJEpnnJHYCoW9QPbQEk2iHsHGGS6qcVUq9HhutG

executed transaction: ca68bb3e931898cdd3c72d6efe373ce26e6845fc486b42bc5d185643ea7a90b1  200 bytes  280 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.bpay","owner":{"threshold":1,"keys":[{"key":"EOS84BLRbGbFahNJEpnnJH...
```

### 시스템 컨트랙트 빌드 및 배포하기

EOSIO 기반 블록체인으로서 기능하려면 다음과 같은 몇 가지 시스템 스마트 컨트랙트를 인스톨 해야 합니다.

* `eosio.contracts`저장소에 있는`eosio.system`, `eosio.msig` ,`eosio.token`
* `eos` 저장소에 있는 `eosio.boot`

먼저 빌드 도구를 설치합니다.

```
sudo apt install build-essential cmake
```

#### eosio.contracts 빌드

eosio.contracts 를 빌드하려면 전용 디렉토리인 eosio.contracts 를 만들고 eosio.contracts 저장소를 클론하여 소스를 빌드합니다. 현재 디렉토리는 pwd로 출력하여 기록해둡니다. 이는 `EOSIO_CONTRACTS_DIRECTORY` 라는 이름으로 참조할 것입니다.

```
cd ~
git clone <https://github.com/EOSIO/eosio.contracts.git>
cd ./eosio.contracts
git checkout release/1.9.x
./build.sh // 빌드 중 eosio.cdt 설치 디렉토리를 묻는데 /usr/opt/eosio.cdt 에 설치되어 있다.
cd ./build/contracts/
pwd
```

#### eosio.boot 시스템 컨트랙트

eos 저장소의 eosio.boot 를 빌드하려면 전용 디렉토리인 eos 를 만들고 eos 저장소를 클론한 뒤 eosio.boot 시스템 계약 소스를 빌드합니다.

```
cd ~
git clone <https://github.com/EOSIO/eos.git>
cd eos
git checkout release/2.1.x
cd ./contracts/contracts/eosio.boot/
cmake
make
pwd
```

pwd 명령으로 출력되는 현재 디렉토리는 EOSIO\_BOOT\_DIRECTORY 라는 이름으로 참조할 것입니다.

#### eosio.token 컨트랙트 배포

이제 eosio.token 컨트랙트를 배포합니다. 이 컨트랙트는 토큰을 발행하거나 정보를 얻거나 이체할 수 있도록 합니다. eosio.token 을 배포하려면 다음과 같이 합니다.

```
//cleos set contract eosio.token EOSIO_CONTRACTS_DIRECTORY/eosio.token/
cleos set contract eosio.token /home/eosuser/eosio.contracts/build/contracts/eosio.token

Reading WAST/WASM from /users/documents/eos/contracts/eosio.token/eosio.token.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 17fa4e06ed0b2f52cadae2cd61dee8fb3d89d3e46d5b133333816a04d23ba991  8024 bytes  974 us
#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1560037f7e7f0060057f7e...
#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...
```

#### eosio.msig 컨트랙트 배포

eosio.msig 는 퍼미션 레벨과 멀티시그 프로세스를 사용할 수 있게 하고 또한 이들을 쉽게 정의하고 관리 할 수 있게 합니다. eosio.msig 컨트랙트를 배포 하려면 다음과 같이 합니다.

```
//cleos set contract eosio.msig EOSIO_CONTRACTS_DIRECTORY/eosio.msig/
cleos set contract eosio.token /home/eosuser/eosio.contracts/build/contracts/eosio.msig

Reading WAST/WASM from /users/documents/eos/build/contracts/eosio.msig/eosio.msig.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 007507ad01de884377009d7dcf409bc41634e38da2feb6a117ceced8554a75bc  8840 bytes  925 us
#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d010000000198011760017f0060047f7e7e7...
#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":{"types":[{"new_type_name":"account_name","type":"name"}],"structs":[{...
```

#### SYS 화폐를 만들고 할당

SYS 화폐를 만들고 토큰을 최대 100억 개 까지 발행할 수 있도록 설정합니다. 그리고 일단 먼저 10억개를 발행해 보도록 하겠습니다. 원한다면 SYS 는 다른 화폐명으로 바꿔도 상관없습니다.

첫 단계로 eosio.token 계정을 사용하여 인증하게 되는 eosio.token 컨트랙트의 create 액션을 사용하여 100억개의 SYS 토큰을 생성합니다. 이는 토큰의 최대 공급량만 설정하는 것으로 실제로 토큰을 발행하지는 않습니다. 발행되지 않은 토큰은 일종의 적립금인 상태라고 생각하면 됩니다.

```
cleos push action eosio.token create '[ "eosio", "10000000000.0000 SYS" ]' -p eosio.token@active

executed transaction: 0440461e0d8816b4a8fd9d47c1a6a53536d3c7af54abf53eace884f008429697  120 bytes  326 us
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"10000000000.0000 SYS"}
```

두 번째로, eosio.token 컨트랙트의 issue 액션을 사용하여 10억 개의 SYS 토큰을 적립된 상태로부터 발행하도록 합니다. 발행한 순간부터 토큰들은 eosio 계정에 귀속됩니다. eosio 계정이 아직 발행되지 않은, 적립된 토큰을 가지고 있기 때문에 액션을 수행하려면 이 계정으로 인증해야 합니다.

```
cleos push action eosio.token issue '[ "eosio", "1000000000.0000 SYS", "memo" ]' -p eosio@active

executed transaction: a53961a566c1faa95531efb422cd952611b17d728edac833c9a55582425f98ed  128 bytes  432 us
#   eosio.token <= eosio.token::issue           {"to":"eosio","quantity":"1000000000.0000 SYS","memo":"memo"}
```

{% hint style="info" %}
경제적 관점에서 보면, 토큰을 발행 함으로서 토큰을 적립금에서 유통되는 상태로 이동시키면, 그로 인해 인플레이션이 발생하게 됩니다. 토큰 발행은 인플레이션을 발생시키는 하나의 방법입니다.
{% endhint %}

#### 시스템 컨트랙트를 배포

v1.8 및 이후 버전에서 도입된 모든 프로토콜 업그레이드 기능을 사용하려면 특수 프로토콜 기능(코드명 PREACTIVATE\_FEATURE)을 활성화하고 이 기능에 의해 도입된 기능을 사용하는, 업데이트된 버전의 시스템 컨트랙트를 배포해야 합니다.

#### PREACTIVATE\_FEATURE 을 활성화

PREACTIVATE\_FEATURE 을 활성화 하려면 다음 코드를 입력합니다.

```
curl --request POST \\
    --url <http://127.0.0.1:8888/v1/producer/schedule_protocol_feature_activations> \\
    -d '{"protocol_features_to_activate": ["0ec7e080177b2c02b278d5088611686b49d739925a92d9bfcacd7fc6b74053bd"]}'

{"result":"ok"}
```

#### eosio.boot 계약 설정

시스템 컨트랙트는 토큰 기반의 행동에 대한 모든 액션을 제공합니다. 시스템 컨트랙트를 배포하기 전 까지는 경제적 운영과는 무관하게 액션이 수행됩니다. 일단 시스템 컨트랙트가 활성화되면 액션을 실행할 때 경제적 요소들을 고려해야 합니다. 즉, 시스템 리소스(CPU, 네트워크, 메모리)에 대한 비용을 지불해야 하며 마찬가지로 계정을 새로 만들 때 비용을 지불해야 합니다. 시스템 컨트랙트를 통해 토큰을 스테이크 및 언스테이크 할 수 있고, 자원을 구입해야 하며, 잠재적 BP를 등록하고 나중에 투표할 수 있으며, BP 보상을 청구할 수 있고, 이런저런 권한과 제약을 설정할 수 있습니다.

이제 eosio.boot 컨트랙트를 설치하면 EOSIO 기반 블록체인에서 매우 권장되는 일련의 프로토콜 기능을 사용할 수 있게 됩니다.

```
//cleos set contract eosio EOSIO_BOOT_DIRECTORY/eosio.boot/
cleos set contract eosio /home/eosuser/eos/contracts/contracts/eosio.boot/build

Reading WAST/WASM from /users/documents/eos/contracts/contracts/eosio.boot/build/eosio.boot.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 2150ed87e4564cd3fe98ccdea841dc9ff67351f9315b6384084e8572a35887cc  39968 bytes  4395 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001be023060027f7e0060067f7e7e7f7f...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"buyrambytes","base":"","fields":[{"name":"p...
```

#### 프로토콜 기능을 활성화

eosio.boot 컨트랙트를 배포 한 후 다음 명령을 사용하여 나머지 기능들을 활성화 합니다.

{% hint style="info" %}
아래 단계는 선택사항이며 이러한 기능 없이도 사용할 수 있지만 EOSIO 기반 블록체인에서는 사용하는것을 적극 권장합니다.
{% endhint %}

```
# KV_DATABASE
```

```
cleos push action eosio activate '["825ee6288fb1373eab1b5187ec2f04f6eacb39cb3a97f356a07c91622dd61d16"]' -p eosio

# ACTION_RETURN_VALUE
cleos push action eosio activate '["c3a6138c5061cf291310887c0b5c71fcaffeab90d5deb50d3b9e687cead45071"]' -p eosio

# CONFIGURABLE_WASM_LIMITS
cleos push action eosio activate '["bf61537fd21c61a60e542a5d66c3f6a78da0589336868307f94a82bccea84e88"]' -p eosio

# BLOCKCHAIN_PARAMETERS
cleos push action eosio activate '["5443fcf88330c586bc0e5f3dee10e7f63c76c00249c87fe4fbf7f38c082006b4"]' -p eosio

# GET_SENDER
cleos push action eosio activate '["f0af56d2c5a48d60a4a5b5c903edfb7db3a736a94ed589d0b797df33ff9d3e1d"]' -p eosio

# FORWARD_SETCODE
cleos push action eosio activate '["2652f5f96006294109b3dd0bbde63693f55324af452b799ee137a81a905eed25"]' -p eosio

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

# ONLY_LINK_TO_EXISTING_PERMISSION
cleos push action eosio activate '["1a99a59d87e06e09ec5b028a9cbb7749b4a5ad8819004365d02dc4379a8b7241"]' -p eosio

# RAM_RESTRICTIONS
cleos push action eosio activate '["4e7bf348da00a945489b2a681749eb56f5de00b900014e137ddae39f48f69d67"]' -p eosio

# WEBAUTHN_KEY
cleos push action eosio activate '["4fca8bd82bbd181e714e283f83e1b45d95ca5af40fb89ad3977b653c448f78c2"]' -p eosio

# WTMSIG_BLOCK_SIGNATURES
cleos push action eosio activate '["299dcb6af692324b899b39f16d5a530a33062804e41f09dc97e9f156b4476707"]' -p eosio
```

#### eosio.system 컨트랙트 배포

다음과 같이 eosio.system 컨트랙트를 배포합니다.

```
# cleos set contract eosio EOSIO_CONTRACTS_DIRECTORY/eosio.system/
cleos set contract eosio /home/eosuser/eosio.contracts/build/contracts/eosio.system/
```

## 단일 제네시스 BP에서 다중 BP로 전환하기

이제 단일 BP(제네시스 노드)에서 다중 BP로 전환하는 방법을 알아보겠습니다. 지금까지는 기본으로 제공된 eosio 계정만 권한이 있으면 블록에 서명할 수 있었습니다. 이번 챕터의 목표는 무엇이 최종 블록인지에 동의하는 2/3+1 BP의 규칙에 따라 운영되는, 선출된 다수의 블록 프로듀서(BP)들이 모여 블록체인을 관리하는 것이 목표입니다.

BP 는 투표에 의해 선정되며 그 결과에 따라 BP 목록이 변경될 수 있습니다. EOSIO의 거버넌스 규칙 하에서는 특정 BP에게 직접 권한을 부여하기 보다는 기본으로 제공되는 eosio.prods 라는 특별한 계정으로 연결됩니다. 이 계정은 선출된 프로듀서 그룹을 나타냅니다. 사실상 BP들의 그룹이나 마찬가지인 eosio.prods 계정은 eosio.msig 계약에 의해 정의된 권한을 운영하고 사용합니다.

eosio.system 컨트랙트를 배포한 후에 eosio.msig 를 특권 있는 계정으로 지정하면 eosio 계정 대신 이 계정으로 권한을 부여할 수 있습니다. 이렇게 하여 eosio 계정은 원래 가지고 있던 권한을 내려놓게 되고 eosio.prods 계정이 그 권한을 대신하게 됩니다.

### 특권 있는 계정으로 eosio.msig 지정

eosio.msig 을 특권 있는 계정으로 지정하려면 다음과 같이 합니다.

```
cleos push action eosio setpriv '["eosio.msig", 1]' -p eosio@active
```

### 시스템 계정 초기화

다음 명령은 system 계정을 코드 제로(초기화 시간에 필요함) 로 초기화하고 SYS 토큰의 정밀도를 4로 지정합니다. 정밀도는 0\~18까지 지정할 수 있습니다.

```
cleos push action eosio init '["0", "4,SYS"]' -p eosio@active
```

### 토큰 스테이킹과 네트워크 확장하기

여기까지 잘 따라 오셨다면 다음과 같은 컨트랙트가 설치된 단일 호스트의, 단일 제네시스 노드 구성이 완료 되었을 것입니다.

* eosio.token
* eosio.msig
* eosio.system

eosio 와 eosio.msig 는 특권을 가진 계정이며 다른 eosio.\* 계정들은 특권을 가진 계정이 아닙니다.

이제 스테이크를 하고 BP 네트워크를 확장 할 준비가 되었습니다.

### 스테이크 된 계정 만들기

스테이크 한다는 것은 EOSIO 시스템 내의 계정에 토큰을 할당하는 프로세스를 의미합니다. 스테이킹과 언스테이킹은 블록체인의 수명 전반에 걸쳐 진행되는 과정입니다만, 바이오스 부트 시퀀스 과정에서 수행되는 초기 스테이킹은 조금 특별합니다. 바이오스 부트 시퀀스 중에는 계정의 토큰이 스테이킹 됩니다. 하지만, BP가 선출되기 전까지 이 토큰은 사실상 동결 상태에 있게 됩니다. 즉, 바이오스 부팅 시퀀스 동안 수행되는 초기 스테이킹의 목표는, 토큰을 계정에 할당하여 사용 가능한 상태로 만들고, 투표 프로세스를 시작하여 BP가 선출되고 블록체인이 "라이브" 상태로 운영되도록 하는 것입니다.

다음은 초기 스테이킹 프로세스에서의 권장 사항입니다.

1. 0.1 토큰(계정 토큰의 10%가 아닌 문자 그대로 0.1 개)이 RAM 용으로 준비됩니다. 기본적으로 cleos 는 계정 생성 시 8KB의 RAM 을 스테이크 하며 이는 계정 생성자가 지불합니다. 초기 스테이킹에서 eosio 계정이 스테이킹을 하는 계정 생성자가 됩니다. 초기 토큰 스테이킹 프로세스 동안 스테이킹된 토큰은 최소 투표 요구 사항이 충족될 때까지는 언스테이킹하여 유동 자산화 할 수 없습니다.
2. CPU의 경우 0.45 토큰이, 네트워크의 경우 0.45 토큰이 스테이크 됩니다.
3. 그 다음으로 사용 가능한 토큰 중 총 9개까지 유동성 토큰으로 보유됩니다.
4. 나머지 토큰은 50/50 으로 CPU 및 네트워크에 할당됩니다.

> 예시 1.\
> accountnum11 계정은 100 SYS 를 가집니다. 이 계정은 0.1000 SYS 를 RAM 에, 45.4500 SYS 를 CPU 에, 45.4500 SYS 를 network 에 스테이킹 합니다. 그리고 9.0000 SYS 를 유동성 토큰으로 가집니다.
>
> 예시 2.\
> accountnum33 계정은 5 SYS 를 가집니다. 이 계정은 0.1000 SYS 를 RAM 에, 0.4500 SYS 를 CPU 에, 0.4500 SYS 를 network 에 스테이킹 합니다. 그리고 4.0000 SYS 를 유동성 토큰으로 가집니다.

이 튜토리얼을 보다 사실적으로 만들려면 10억 개의 토큰을 계정에 배포하고 파레토 분포를 사용할 수 있습니다. 파레토 분포는 80-20 규칙을 모델링 한 것입니다. 예를 들어, 토큰의 80%를 토큰 보유자의 20%가 가지는 것입니다. 이 예제에서는 분포를 생성하는 방법은 다루지 않을 것이며 스테이킹을 수행하는 명령에 초점을 맞추고 있습니다. 이 튜토리얼에서 제공되는 스크립트 [bios-boot-tutorial.py](http://bios-boot-tutorial.py) 는 Python NumPy(numpy) 라이브러리를 사용하여 파레토 분포를 생성할 수 있습니다.

다음 단계를 따라 각 계정에서 토큰을 스테이크 합니다. 이 단계들은 각각의 계정마다 따로 수행해야 합니다.

{% hint style="info" %}
이 튜토리얼에서는 아래의 키 쌍을 사용할 것입니다. 실제 프로덕션 환경에서 계정의 키 값과 토큰은 잘 정의된 프로세스를 통해 이미 공유되어 있어야 합니다.
{% endhint %}

```
cleos create key --to-console
Private key: 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
Public key: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt

cleos wallet import --private-key 5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o
imported private key for: EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt
```

초기 리소스와 공개키로 스테이크된 계정을 생성합니다.

```
cleos system newaccount eosio --transfer accountnum11 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt --stake-net "100000000.0000 SYS" --stake-cpu "100000000.0000 SYS" --buy-ram-kbytes 8192

775292ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"0000000000ea30551082d4334f4d113200200000"} arg: {"code":"eosio","action":"buyrambytes","args":{"payer":"eosio","receiver":"accountnum11","bytes":8192}}
775295ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"0000000000ea30551082d4334f4d113200ca9a3b00000000045359530000000000ca9a3b00000000045359530000000001"} arg: {"code":"eosio","action":"delegatebw","args":{"from":"eosio","receiver":"accountnum11","stake_net_quantity":"100000.0000 SYS","stake_cpu_quantity":"100000.0000 SYS","transfer":true}}
executed transaction: fb47254c316e736a26873cce1290cdafff07718f04335ea4faa4cb2e58c9982a  336 bytes  1799 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"accountnum11","owner":{"threshold":1,"keys":[{"key":"EOS8mUftJXepGzdQ2TaC...
#         eosio <= eosio::buyrambytes           {"payer":"eosio","receiver":"accountnum11","bytes":8192}
#         eosio <= eosio::delegatebw            {"from":"eosio","receiver":"accountnum11","stake_net_quantity":"100000.0000 SYS","stake_cpu_quantity...
```

### 새로운 계정을 BP 로 등록하기

새로운 계정을 BP로 등록하려면 다음과 같이 합니다다.

```
cleos system regproducer accountnum11 EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt <https://accountnum11.com> 401

1487984ms thread-0   main.cpp:419                  create_action        ] result: {"binargs":"1082d4334f4d11320003fedd01e019c7e91cb07c724c614bbf644a36eff83a861b36723f29ec81dc9bdb4e68747470733a2f2f6163636f756e746e756d31312e636f6d2f454f53386d5566744a586570477a64513254614364754e7553504166584a48663232756578347534316162314556763945416857740000"} arg: {"code":"eosio","action":"regproducer","args":{"producer":"accountnum11","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","url":"<https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","location>":0}}
executed transaction: 4ebe9258bdf1d9ac8ad3821f6fcdc730823810a345c18509ac41f7ef9b278e0c  216 bytes  896 us
#         eosio <= eosio::regproducer           {"producer":"accountnum11","producer_key":"EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt","u...
```

위 명령은 계정을 BP 후보로 만듭니다. 하지만 투표를 받아 선출되지 않는 이상 아직 블록을 생성하지는 않습니다.

### BP 목록 보기

투표 프로세스를 수월하게 하기 위해 다음 명령으로 BP 목록을 볼 수 있습니다. 현 시점에서는 BP로 등록된 계정만이 나타날 것입니다.

```
cleos system listproducers

Producer      Producer key                                           Url                                                         Scaled votes
accountnum11  EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt  <https://accountnum11.com/EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22> 0.0000
```

### 추가 BP 설정 및 시작

추가 BP를 설정하고 이전에 만든 accountnum11 계정을 사용해 보겠습니다. 추가 BP를 설정하려면 다음과 같이 수행합니다.

```
cd ~/biosboot/
mkdir accountnum11
cd accountnum11
copy ~/biosboot/genesis/stop.sh
copy ~/biosboot/genesis/clean.sh
```

다음과 같은 3개의 쉘 스크립트를 만들고 실행 권한을 줍니다.

`genesis_start.sh`

```
#!/bin/bash
DATADIR="./blockchain"
CURDIRNAME=${PWD##*/}

if [ ! -d $DATADIR ]; then
  mkdir -p $DATADIR;
fi

nodeos \\
--genesis-json $DATADIR"/../../genesis.json" \\
--signature-provider EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt=KEY:5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o \\
--plugin eosio::producer_plugin \\
--plugin eosio::producer_api_plugin \\
--plugin eosio::chain_plugin \\
--plugin eosio::chain_api_plugin \\
--plugin eosio::http_plugin \\
--plugin eosio::history_api_plugin \\
--plugin eosio::history_plugin \\
--data-dir $DATADIR"/data" \\
--blocks-dir $DATADIR"/blocks" \\
--config-dir $DATADIR"/config" \\
--producer-name $CURDIRNAME \\
--http-server-address 127.0.0.1:8011 \\
--p2p-listen-endpoint 127.0.0.1:9011 \\
--access-control-allow-origin=* \\
--contracts-console \\
--http-validate-host=false \\
--verbose-http-errors \\
--enable-stale-production \\
--p2p-peer-address localhost:9010 \\
--p2p-peer-address localhost:9012 \\
--p2p-peer-address localhost:9013 \\
>> $DATADIR"/nodeos.log" 2>&1 & \\
echo $! > $DATADIR"/eosd.pid"
```

`start.sh`

```
#!/bin/bash
DATADIR="./blockchain"
CURDIRNAME=${PWD##*/}

if [ ! -d $DATADIR ]; then
  mkdir -p $DATADIR;
fi

nodeos \\
--signature-provider EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt=KEY:5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o \\
--plugin eosio::producer_plugin \\
--plugin eosio::producer_api_plugin \\
--plugin eosio::chain_plugin \\
--plugin eosio::chain_api_plugin \\
--plugin eosio::http_plugin \\
--plugin eosio::history_api_plugin \\
--plugin eosio::history_plugin \\
--data-dir $DATADIR"/data" \\
--blocks-dir $DATADIR"/blocks" \\
--config-dir $DATADIR"/config" \\
--producer-name $CURDIRNAME \\
--http-server-address 127.0.0.1:8011 \\
--p2p-listen-endpoint 127.0.0.1:9011 \\
--access-control-allow-origin=* \\
--contracts-console \\
--http-validate-host=false \\
--verbose-http-errors \\
--enable-stale-production \\
--p2p-peer-address localhost:9010 \\
--p2p-peer-address localhost:9012 \\
--p2p-peer-address localhost:9013 \\
>> $DATADIR"/nodeos.log" 2>&1 & \\
echo $! > $DATADIR"/eosd.pid"
```

`hard_start.sh`

```
#!/bin/bash
DATADIR="./blockchain"
CURDIRNAME=${PWD##*/}

if [ ! -d $DATADIR ]; then
  mkdir -p $DATADIR;
fi

nodeos \\
--signature-provider EOS8mUftJXepGzdQ2TaCduNuSPAfXJHf22uex4u41ab1EVv9EAhWt=KEY:5K7EYY3j1YY14TSFVfqgtbWbrw3FA8BUUnSyFGgwHi8Uy61wU1o \\
--plugin eosio::producer_plugin \\
--plugin eosio::producer_api_plugin \\
--plugin eosio::chain_plugin \\
--plugin eosio::chain_api_plugin \\
--plugin eosio::http_plugin \\
--plugin eosio::history_api_plugin \\
--plugin eosio::history_plugin \\
--data-dir $DATADIR"/data" \\
--blocks-dir $DATADIR"/blocks" \\
--config-dir $DATADIR"/config" \\
--producer-name $CURDIRNAME \\
--http-server-address 127.0.0.1:8011 \\
--p2p-listen-endpoint 127.0.0.1:9011 \\
--access-control-allow-origin=* \\
--contracts-console \\
--http-validate-host=false \\
--verbose-http-errors \\
--enable-stale-production \\
--p2p-peer-address localhost:9010 \\
--p2p-peer-address localhost:9012 \\
--p2p-peer-address localhost:9013 \\
--hard-replay-blockchain \\
>> $DATADIR"/nodeos.log" 2>&1 & \\
echo $! > $DATADIR"/eosd.pid"
```

오류 없이 모든 과정이 제대로 수행된다면 다음과 같은 폴더가 보일 것입니다.

```
cd ~/biosboot/accountnum11/
ls -al
drwxr-xr-x   8 owner  group   256 Dec  7 14:17 .
drwxr-xr-x   3 owner  group   960 Dec  5 10:00 ..
-rwxr-xr-x   1 owner  group   40  Dec  5 13:08 clean.sh
-rwxr-xr-x   1 owner  group   947 Dec  5 14:31 genesis_start.sh
-rwxr-xr-x   1 owner  group   888 Dec  5 13:08 hard_start.sh
-rwxr-xr-x   1 owner  group   901 Dec  6 15:44 start.sh
-rwxr-xr-x   1 owner  group   281 Dec  5 13:08 stop.sh
```

다음 명령으로 두 번째 BP노드를 시작합니다.

```
cd ~/biosboot/accountnum11/
./genesis_start.sh
tail -f blockchain/nodeos.log
```

위 명령을 실행한 후 nodeos 가 생성하는 로그 스트림이 nodeos.log 에 출력되는 것을 콘솔에서 볼 수 있습니다. CTRL+C 키를 누르면 로그 스트림의 콘솔 출력을 중지합니다.

새 노드를 중지하려면 [stop.sh](http://stop.sh) 를 실행합니다. 노드를 다시 시작하려면 genesis\_start.sh가 아니라 [start.sh](http://start.sh/) 스크립트를 실행합니다. (genesis\_start.sh 는 제네시스 노드 시작 시 한 번만 사용합니다).

모든 것을 지우고 처음부터 다시 시작하려면 다음 명령을 실행합니다.

```
cd ~/biosboot/accountnum11/
./stop.sh
./clean.sh
./genesis_start.sh
tail -f blockchain/nodeos.log
```

### 여러 대의 BP 노드 만들기

이제 [여기](undefined-5.md#2.4)서 [여기](undefined-5.md#2.6-bp)까지 과정을 반복하여 각각 자체 스테이크 계정, 자체 전용 디렉토리와 accountnumXY 라고 명명된 계정, 자체 전용 스크립트 파일(genesis\_start.sh, [start.sh](http://start.sh/), [stop.sh](http://stop.sh/), [clean.sh](http://clean.sh))을 사용하여 원하는 만큼 BP를 생성할 수 있습니다.

또한 만들어낸 노드들을 서로 연결할 수 있어야 합니다. 다음 매개 변수에 주의하여 genesis\_start.sh, [start.sh](http://start.sh/) 및 hard\_start.sh 스크립트를 작성합니다.

```
--producer-name $CURDIRNAME \\ # Producer name, set in the script to be the parent directory name
...
--http-server-address 127.0.0.1:8011 \\ # http listening port for API incoming requests
--p2p-listen-endpoint 127.0.0.1:9011 \\ # p2p listening port for incoming connection requests
...
...
--p2p-peer-address localhost:9010 \\   # Meshing with peer `genesis` node
--p2p-peer-address localhost:9012 \\   # Meshing with peer `accountnum12` node
--p2p-peer-address localhost:9013 \\.  # Meshing with peer `accountnum13` node
```

### 새로 생성한 BP에게 투표하기

이제 여러 노드들이 시작되었고 이들이 연결된 네트워크가 만들어 졌습니다. 그러나 제네시스 노드로 부터 블록을 전달받고는 있지만 아직 스스로 블록을 생성하고 있지는 않습니다.

#### 15% 요구사항

노드가 블록을 생성하려면 토큰 공급량의 총 15%를 스테이크한 후 준비가 된 모든 BP 들에게 투표해야 합니다. accountnum11 계정에는 이미 이전에 할당된 토큰이 충분히 있습니다. BP 를 더 선출하려면 다음 명령을 실행합니다. 한 계정은 계정 이름으로 식별 가능한 최대 30명의 BP 에게 투표할 수 있습니다.

```
cleos system voteproducer prods accountnum11 accountnum11 accountnum12 accountnum13
```

## eosio 계정 파기와 시스템 계정

BP 가 선출되고 최소한 15% 의 토큰이 투표를 위해 스테이킹 되어 있어야 한다 라는 최소한의 요구사항이 만족된 후, eosio 는 파기할 수 있습니다. 그러면 eosio.msig 만이 유일하게 특권을 가진 계정이 됩니다.

eosio.\* 계정을 파기하려면 eosio.\* 의 키 세트를을 null 로 설정하면 됩니다. 다음 명령을 사용하여 eosio.\* 계정의 owner 키와 active 키를 삭제합니다.

```
cleos push action eosio updateauth '{"account": "eosio", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@owner
cleos push action eosio updateauth '{"account": "eosio", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@active
```

필요시한 [이 단계](undefined-5.md#undefined-10)에서 만든 시스템 계정들 역시 다음 명령을 사용하여 파기할 수 있습니다.

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

## 모니터링하고 테스트하기

실행중인 nodeos(제네시스 노드와 BP 노드) 를 다음과 같이 모니터링 할 수 있습니다.

```
cd ~/biosboot/genesis/
tail -f ./blockchain/nodeos.log

cd ~/biosboot/accountnum11/
tail -f ./blockchain/nodeos.log
```

이제 여러 명령이나 계정 생성, 잔고 확인, 토큰 전송등을 테스트 할 수 있습니다.

새 계정을 만들려면 Create test account 튜토리얼을 확인합니다.

토큰을 발행하고 할당하거나고 계정간에 전송하려면 Deploy, Issue,and Transfer 섹션을 확인 합니다.



## Reference

&#x20;__ [_BIOS Boot Sequence | EOSIO Developer Docs_](https://developers.eos.io/welcome/latest/tutorials/bios-boot-sequence/?query=bios%20boot%20sequence\&page=1#gatsby-focus-wrapper)
