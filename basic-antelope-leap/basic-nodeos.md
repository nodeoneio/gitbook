# Nodeos 기초

## 개요

[이전 단원](install-leap-software.md)에서 Leap  소프트웨어를 설치하였습니다. 본격적으로 뭔가 시작해보기 전에 Leap 의 핵심 컴포넌트들에 대한 기초적인 내용을 먼저 짚어보겠습니다.

Antelope 노드의 핵심 데몬(daemon)인 `nodeos` 는 CLI(명령줄 인터페이스) 환경에서 실행하는 응용 프로그램입니다. `nodeos` 는 명령줄에서 직접 실행해도 되지만 보통 노드 운영 환경에서는 `nodeos` 실행에 필요한 조건과 옵션들을 bash 나 Python 등의 스크립트로 만들어 실행합니다.

`nodeos` 을 실행할 때 일반적으로 여러가지 옵션이 필요합니다. 옵션은 명령줄에 직접 입력하거나 환경설정 파일에 미리 구성해 놓을 수 있습니다. `nodeos` 프로그램에서 사용 가능한 옵션은 다음과 같이 두 가지 종류로 나눌 수 있습니다.&#x20;

* [nodeos 전용 옵션](basic-nodeos.md#nodeos)\
  `nodoes` 프로그램 데몬 자체의 동작을 제어하는 옵션이며 명령줄에서만 입력할 수 있습니다.
* [nodeos 플러그인 전용 옵션](basic-nodeos.md#nodeos-1)\
  `nodeos` 플러그인 전용 옵션은 `nodeos` 플러그인의 동작을 제어하며, `nodeos` 실행 시 명령줄 옵션으로 입력하거나 `config.ini` 파일 안에 설정할 수 있습니다. 어떤 옵션들을 설정하는가에 따라 Antelope 노드의 기능이 달라집니다.

`nodeos` 옵션에 대한 자세한 정보는 `nodeos --help` 를 실행하면 확인할 수 있습니다.

## nodeos 전용 옵션

`nodeos` 전용 옵션은 블록체인 데이터가 있는 디렉토리 설정, `nodeos` 설정 파일 이름 지정, 로깅 설정 파일의 이름과 경로 설정 등과 같이, 주로 `nodeos` 프로그램 자체 설정을 위한 용도로 사용됩니다.

`nodeos --help` 를 입력하면 도움말이 출력되는데, 출력된 내용의 하단에서 다음과 같이 `nodeos` 전용 옵션들을 확인 할 수 있습니다.

{% code overflow="wrap" %}
```
$ nodeos --help
...
....
...
Application Config Options:
  --plugin arg                          사용할 플러그인을 지정합니다. 여러 개를 지정할 수 있습니다.

Application Command Line Options:
  -h [ --help ]                         도움말을 출력하고 종료합니다.
  -v [ --version ]                      버전 정보를 출력합니다.
  --full-version                        모든 버전 정보를 출력합니다.
  --print-default-config                디폴트 설정 템플릿을 출력합니다.
  -d [ --data-dir ] arg                 프로그램 런타임 데이터가 들어있는 디렉토리를 지정합니다.
  --config-dir arg                      config.ini 와 같은 설정 파일이 있는 디렉토리를 
                                        지정합니다.
  -c [ --config ] arg (=config.ini)     config-dir에 저장된 설정 파일 이름을 지정합니다.
  -l [ --logconf ] arg (=logging.json)  라이브러리 사용자를 위한 로깅 구성 파일/경로를 지정합니다. 
```
{% endcode %}

## nodeos 플러그인 전용 옵션

`nodeos` 플러그인 전용 옵션은 `nodeos` 플러그인의 동작을 제어합니다. 플러그인 전용 옵션마다 이름이 다르기 때문에 순서에 상관없이 명령줄이나 `config.ini` 파일 안에 원하는 방식대로 기입할 수 있습니다. 일반적으로 어떤 플러그인과 그 플러그인의 옵션들을 한 데 묶어 기입하는 것이 관리하기 편리합니다.&#x20;

어떤 플러그인 전용 옵션을 설정할 때는 반드시 `--plugin` 옵션으로 이 옵션이 적용될 플러그인도 함께 사용하도록 설정되어 있는지 확인해야 합니다. 그렇지 않으면 플러그인 전용 옵션은 무시됩니다.

각 플러그인별 전용 옵션에 대한 자세한 내용은 [Nodeos Plugin 상세](install-leap-software/nodeos-plugin-details/) 단원에서 찾아볼 수 있습니다.

대부분의 `config.ini` 옵션에는 그에 대응하는 CLI 옵션이 존재합니다. 예를 들어 CLI 옵션 `--plugin eosio::chain_api_plugin` 은 `config.ini` 의 `plugin = eosio::chain_api_plugin` 에 대응됩니다.

몇몇 CLI 옵션은 `config.ini` 에서 사용할 수 없습니다. 예를 들어 `state_history_plugin` 의 `--delete-state-history` 옵션이나 `chain_plugin` 의 `--genesis-json` 과 같은 플러그인 전용 옵션은 `config.ini`에서 사용할 수 없습니다.&#x20;

`nodeos --help` 실행하면 아래 예시와 같이 `Command Line Options for...` 라는 섹션에서 이러한 명령줄 전용 옵션들을 찾아볼 수 있습니다.

{% code overflow="wrap" %}
```
$ nodeos --help
....
Command Line Options for eosio::chain_plugin:
  --genesis-json arg                    제네시스 상태를 읽어들일 파일
  --genesis-timestamp arg               제네시스 상태 파일의 초기 타임스탬프를 덮어씌움.
...
```
{% endcode %}

## **config.ini 설정 파일 위치**

`config.ini` 는 노드의 동작과 역할을 제어하는 환경 설정 파일입니다. 디폴트로 `config.ini` 는  `~/.local/share/eosio/nodeos/config` 디렉토리에 위치합니다.

직접 작성한 `config.ini` 파일을 사용하려면, `nodeos` 실행 시 명령줄에서 `-config <path>/config.ini` 옵션을 설정하면 됩니다. 환경 설정 파일은 `nodeos` 데이터 디렉토리와 같이 쉽게 접근할 수 있는 곳에 저장하는 것이 사용하기 편리합니다.

## nodeos 실행 <a href="#nodeos_run_example" id="nodeos_run_example"></a>

이제 `nodeos` 를 실행해 볼 수 있습니다.아직 Leap 를 설치하지 않았다면 [Leap 소프트웨어 설치 단원](install-leap-software.md) 참조하여 Leap 를 설치합니다.&#x20;

다음은 로컬 환경에서 단독으로 실행되는 블록 생산자(BP)의 노드를 시작하는 명령입니다. 명령줄에서 실행합니다.&#x20;

{% code overflow="wrap" %}
```bash
$ nodeos \
-e -p eosio \
--data-dir /home/nodeos \
--config-dir /home/nodeos/ \
--plugin eosio::producer_plugin \
--plugin eosio::chain_plugin \
--plugin eosio::http_plugin \
--plugin eosio::chain_api_plugin \
--http-server-address 0.0.0.0:8888 \
--p2p-listen-endpoint 0.0.0.0:9876 \
--contracts-console \
--disable-replay-opts \
--access-control-allow-origin='*' \
--http-validate-host=false \
--verbose-http-errors \
>> nodeos.log 2>&1 &
```
{% endcode %}

위 예제에서 `nodeos` 명령과 함께 사용된 옵션은 다음과 같습니다.

* 블록 생성자로 지정하여 블록을 생성하도록 합니다. (`e`)
* 블록 생성자 이름을 "eosio"  로 지정합니다. (`p`)
* data 디렉토리를 지정합니다. (`-data-dir`) data 디렉토리는 블록체인의 블록 데이터를 저장할 루트 디렉토리입니다.
* `config.ini` 가 위치한 디렉토리를 지정합니다. (`-config-dir`)
* `-plugin` 으로 다음 플러그인들을 로딩합니다. \
  `producer_plugin`, `chain_plugin`, `http_plugin`, `chain_api_plugin`
* 다음과 같은 `chain_plugin` 의 전용 옵션들을 설정합니다.\
  `-contracts-console`, `-disable-replay-opts`
* 다음과 같은 `http-plugin`의 전용 옵션들을 설정합니다.\
  `-access-control-allow-origin`, `-http-validate-host`, `-verbose-http-errors`
* `stdout` 와 `stderr` 로 출력되는 로그를 `nodeos.log` 파일로 전달합니다. (`nodeos.log 2>&1`)
* 프로세스를 백그라운드로 돌리고 쉘 프롬프트로 돌아옵니다. (`&`)

## nodeos 동작 확인

이제 정상적으로 `nodeos` 가 동작하고 있는지 확인해 봅시다. 노드의 정상 동작 여부는 다음 두 가지 방법으로 확인할 수 있습니다.

* [cleos 명령 실행](basic-nodeos.md#cleos)
* [출력되는 로그 확인](basic-nodeos.md#undefined-2)

### cleos 명령 실행

`cleos get info` 명령을 실행하면 다음과 같이 명령을 실행한 순간의 로컬 노드 정보가 콘솔에 출력됩니다. 이 정보로 노드의 상태가 정상인지 확인할 수 있습니다.

{% code overflow="wrap" %}
```
$ cleos get info
{
  "server_version": "579d3f65",
  "chain_id": "8a34ec7df1b8cd06ff4a8abbaa7cc50300823350cadc59ab296cb00d104d2b8f",
  "head_block_num": 50,
  "last_irreversible_block_num": 49,
  "last_irreversible_block_id": "00000031970ed16f014146449729c4c3b9a396b83f14d153821062cf6d4306ac",
  "head_block_id": "000000324d365da43cca85cf200d7c1b8b3b37d5376c2f05f2fec5cf4dde6824",
  "head_block_time": "2022-08-22T14:22:40.500",
  "head_block_producer": "leap",
  "virtual_block_cpu_limit": 210026,
  "virtual_block_net_limit": 1101237,
  "block_cpu_limit": 200000,
  "block_net_limit": 1048576,
  "server_version_string": "v3.1.0",
  "fork_db_head_block_num": 50,
  "fork_db_head_block_id": "000000324d365da43cca85cf200d7c1b8b3b37d5376c2f05f2fec5cf4dde6824",
  "server_full_version_string": "v3.1.0-rc4-579d3f659b2dda2d2a515dd7bd866770f93f4e0d",
  "total_cpu_weight": 0,
  "total_net_weight": 0,
  "earliest_available_block_num": 1,
  "last_irreversible_block_time": "2022-08-22T14:22:40.000"
}
```
{% endcode %}

### 출력되는 로그 확인

위 명령에서 `nodeos` 가 출력하는 `stdout`, `stderr` 로그를 `nodeos.log` 에 기록하도록 하였습니다. 다음 명령으로 노드가 출력하는 로그를 실시간으로 확인할 수 있습니다.

{% code overflow="wrap" %}
```
$ tail -f nodeos.log
...
info  2022-08-22T14:34:18.403 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block f27f135f555c20aa... #7 @ 2022-08-22T14:34:18.500 signed by eosio [trxs: 0, lib: 6, confirmed: 0]
info  2022-08-22T14:34:18.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block 76c79aac6ddd6f3b... #8 @ 2022-08-22T14:34:19.000 signed by eosio [trxs: 0, lib: 7, confirmed: 0]
info  2022-08-22T14:34:19.401 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block a8ddf261459fc702... #9 @ 2022-08-22T14:34:19.500 signed by eosio [trxs: 0, lib: 8, confirmed: 0]
info  2022-08-22T14:34:19.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block 55c20a35f5553d4f... #10 @ 2022-08-22T14:34:20.000 signed by eosio [trxs: 0, lib: 9, confirmed: 0]
info  2022-08-22T14:34:20.401 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block f261faahjf7tjfi8... #11 @ 2022-08-22T14:34:20.500 signed by eosio [trxs: 0, lib: 10, confirmed: 0]
info  2022-08-22T14:34:21.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block ndj573hyd7oedjdj... #12 @ 2022-08-22T14:34:21.000 signed by eosio [trxs: 0, lib: 11, confirmed: 0]
...
```
{% endcode %}

정상적으로 `nodeos` 가 동작 중이라면 위와 같이 로그에서 블록이 생성되고 있는 과정을 확인할 수 있습니다.

여기까지 로컬에서 단독으로 실행되는 Antelope 블록체인 노드 만들어 보았습니다. 일단 이 단계까지 진행되었으면 실습용으로 사용할 최소한의 조건을 가진 노드가 만들어 진 것입니다. `nodeos` 에서 사용하는 플러그인과 옵션에 대한 상세한 내용이 필요하면 [nodeos Plugin 상세](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/mhCOoiYoXaVjjMMGXHdI/basic-antelope-leap/nodeos-plugin-details) 단원을 참조합니다.

## nodeos 중지하기

`nodeos` 를 중지하려면 먼저 nodeos 의 PID 를 찾은 다음, 인터럽트 시그널인 `SIGINT` 를 전달하면 됩니다. `SIGINT` 는 디폴트 시그널이기 때문에 단순히 `kill <PID>`  명령으로 데몬을 제거할 수 있습니다.

다음은 `nodeos` 데몬을 제거하는 예제입니다.

{% code overflow="wrap" %}
```
$ ps
...
500 pts/2    00:19:46 nodeos
...
$ kill 500
```
{% endcode %}

만약 `kill -SIGKILL <PID>` 명령으로 데몬을 강제 종료하면 블록체인 상태 데이터베이스가 망가져, 블록체인 데이터를 지우고 다시 처음부터 시작하거나 리플레이를 해야 합니다. 따라서 블록체인을 나중에 다시 계속 이어서 사용하려면 강제 종료는 사용하지 않는 것이 좋습니다.&#x20;

중지했던 노드를 다시 시작하려면 앞서 입력했던 `nodeos` 실행 명령을 다시 실행하면 됩니다.

만약 `nodeos` 를 강제 종료하여 노드 데이터베이스가 깨져 `nodeos` 가 재기동 할 수 없는 상황이라면 다음 명령으로 블록 로그와 상태 데이터베이스 파일을 지운 뒤 `nodeos` 를 실행하면 처음부터 다시 블록을 만들 수 있습니다.

```
rm -rf /home/nodeos/blocks/*
rm -rf /home/nodeos/state/*
```

## 환경설정 파일을 이용하여 노드 시작하기

실행한 Antelope 노드를 장기적으로 사용할 계획이라면 환경설정 파일과 시작/중지 스크립트를 작성하여 보다 쉽게 노드를 운영할 수 있습니다. 이 내용은 [바이오스 부트 시퀀스 문서](bios-boot-sequence.md)의 제네시스 노드 구성 단원에서 상세히 다루고 있습니다.

처음으로 Antelope 을 학습하는 단계라면 우선은 명령줄에서 `nodeos` 를 실행하는 방법을 사용하면서 학습해 나가도록 합시다.
