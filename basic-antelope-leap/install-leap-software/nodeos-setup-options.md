# Nodeos 기초

## 개요

Antelope 노드의 핵심 데몬(daemon)인 nodeos 는 CLI(명령줄 인터페이스) 환경에서 실행할 수 있는 응용 프로그램입니다. nodeos 는 명령줄에서 직접 실행하거나 별도로 자동화된 스크립트를 만들어 실행 할 수 있습니다.&#x20;

Antelope 노드가 어떠한 기능을 하는가는 설정한 nodeos 플러그인 옵션에 따라 결정됩니다. nodeos 프로그램에서 사용 가능한 옵션은 다음과 같이 두 가지 종류로 나눌 수 있습니다.&#x20;

* nodeos 전용 옵션: \
  nodoes 프로그램 데몬 자체의 동작을 제어하는 옵션이며 명령줄에서만 입력할 수 있습니다.
* nodeos 플러그인 전용 옵션: \
  nodeos 플러그인 전용 옵션은 nodeos 플러그인의 동작을 제어하며,  nodeos 실행 시 명령줄 옵션으로 입력하거나 `config.ini` 파일 안에 설정할 수 있습니다.&#x20;

&#x20;nodeos 옵션에 대한 자세한 정보는 `nodeos --help` 를 실행하면 확인할 수 있습니다.

## nodeos 전용 옵션

nodeos 전용 옵션은 블록체인 데이터가 있는 디렉토리 설정, nodeos 설정 파일 이름 지정, 로깅 설정 파일의 이름과 경로 설정 등, 주로 nodeos 프로그램 자체 설정을 위한 용도로 사용됩니다. `nodeos --help` 를 입력하면 도움말이 출력되는데, 다음과 같이 출력된 내용의 하단에서 nodeos 전용 옵션들을 확인 할 수 있습니다.

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

## nodeos 플러그인 전용 옵션

nodeos 플러그인 전용 옵션은 nodeos 플러그인의 동작을 제어합니다. 플러그인 전용 옵션마다 고유한 이름이 있기 때문에 순서에 상관없이 명령줄이나 `config.ini` 파일 안에 원하는 방식대로 기입할 수 있습니다. 일반적으로 어떤 플러그인과 그 플러그인의 옵션들을 한 데 묶어 기입하는 것이 관리하기 편리합니다.&#x20;

어떤 플러그인 전용 옵션을 설정할 때는 반드시 `--plugin` 옵션을 사용하여, 이 옵션이 적용될 플러그인도 함께 사용하도록 설정되어 있는지 확인해야 합니다. 그렇지 않으면 이 플러그인 전용 옵션은 무시됩니다.

각 플러그인별 전용 옵션에 대한 자세한 내용은 [Nodeos Plugin 상세](nodeos-plugin-details/) 단원에서 찾아볼 수 있습니다.

대부분의 `config.ini` 옵션에는 그에 대응하는 CLI 옵션이 존재합니다. 예를 들어 CLI 옵션 `--plugin eosio::chain_api_plugin` 은 `config.ini` 의 `plugin = eosio::chain_api_plugin` 에 대응됩니다.

몇몇 CLI 옵션은 `config.ini` 에서 사용할 수 없습니다. 예를 들어 `state_history_plugin` 의 `--delete-state-history` 옵션이나 `chain_plugin` 의 `--genesis-json` 과 같은 플러그인 전용 옵션은 `config.ini`에서 사용할 수 없습니다.&#x20;

`nodeos --help` 실행하면 아래 예시와 같이 `Command Line Options for...` 라는 섹션에서 이러한 옵션들을 찾아볼 수 있습니다.

```
$ nodeos --help
....
Command Line Options for eosio::chain_plugin:
  --genesis-json arg                    제네시스 상태를 읽어들일 파일
  --genesis-timestamp arg               제네시스 상태 파일의 초기 타임스탬프를 덮어씌움.
...
```

## **config.ini 설정 파일 위치**

디폴트로 `config.ini` 는  `~/.local/share/eosio/nodeos/config` 폴더에 위치합니다.

직접 작성한 `config.ini` 파일을 사용하려면, `nodeos` 실행 시 명령줄에서 `-config <path>/config.ini` 옵션을 설정하면 됩니다.

## nodeos 실행 예제

이제 nodeos 를 실행해 봅시다. 다음은 로컬 환경에서 단독으로 실행되는 블록 생산자(BP)의 노드를 시작할 때의 nodeos 실행 예제입니다.

```bash
$ nodeos \\
  -e -p leap \\
  --data-dir /users/blockchain/antelope/data     \\
  --config-dir /users/blockchain/antelope/config \\
  --plugin eosio::producer_plugin      \\
  --plugin eosio::chain_plugin         \\
  --plugin eosio::http_plugin          \\
  --plugin eosio::state_history_plugin \\
  --contracts-console   \\
  --disable-replay-opts \\
  --access-control-allow-origin='*' \\
  --http-validate-host=false        \\
  --verbose-http-errors             \\
  --state-history-dir /shpdata \\
  --trace-history              \\
  --chain-state-history        \\
  >> nodeos.log 2>&1 &
```

위 예제에서 `nodeos` 명령에 주어진 옵션은 다음과 같습니다.

* 블록 생성자로 지정하여 블록을 생성하도록 합니다. (`e`)
* 블록 생성자 이름을 "leap" 로 지정합니다. (`p`)
* data 디렉토리를 지정합니다. (`-data-dir`) data 디렉토리는 블록체인의 블록 데이터를 저장할 Root 디렉토리입니다.
* `config.ini` 가 위치한 디렉토리를 지정합니다. (`-config-dir`)
* `-plugin` 으로 다음 플러그인들을 로딩합니다. \
  `producer_plugin`, `chain_plugin`, `http_plugin`, `state_history_plugin`
* 다음과 같은의 `chain_plugin` 의 전용 옵션들을 설정합니다.\
  `-contracts-console`, `-disable-replay-opts`
* 다음과 같은 `http-plugin`의 전용 옵션들을 설정합니다.\
  `-access-control-allow-origin`, `-http-validate-host`, `-verbose-http-errors`
* 다음과 같은 `state_history_plugin`의 전용 옵션들을 설정합니다.\
  `-state-history-dir`, `-trace-history`, `-chain-state-history`
* `stdout` 와 `stderr` 로 출력되는 로그를 `nodeos.log` 파일로 전달합니다. (`nodeos.log 2>&1`)
* 프로세스를 백그라운드로 돌리고 쉘 프롬프트로 돌아옵니다. (`&`)

## nodeos 동작 확인

이제 정상적으로 nodeos 가 동작하고 있는지 확인해 봅시다. 노드의 정상 동작 여부는 다음 두 가지 방법으로 확인할 수 있습니다.

* `cleos` 명령 실행
* 출력되는 로그 확인

### cleos 명령 실행

로컬 노드의 정보를 확인할 수 있는 `cleos get info` 명령을 실행하면 다음과 같이 명령을 실행한 순간의 정보가 콘솔에 출력됩니다. 이 정보로 노드의 상태가 정상인지 확인할 수 있습니다.

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

### 출력되는 로그 확인

위 예제에서 nodeos 가 출력하는 stdout, stderr 로그를 nodeos.log 에 기록하도록 설정하였습니다. 따라서 다음 명령으로 노드의 동작 상황을 확인할 수 있습니다.

```
$ tail -f nodeos.log
...
info  2022-08-22T14:34:18.403 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block f27f135f555c20aa... #7 @ 2022-08-22T14:34:18.500 signed by leap [trxs: 0, lib: 6, confirmed: 0]
info  2022-08-22T14:34:18.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block 76c79aac6ddd6f3b... #8 @ 2022-08-22T14:34:19.000 signed by leap [trxs: 0, lib: 7, confirmed: 0]
info  2022-08-22T14:34:19.401 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block a8ddf261459fc702... #9 @ 2022-08-22T14:34:19.500 signed by leap [trxs: 0, lib: 8, confirmed: 0]
info  2022-08-22T14:34:19.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block 55c20a35f5553d4f... #10 @ 2022-08-22T14:34:20.000 signed by leap [trxs: 0, lib: 9, confirmed: 0]
info  2022-08-22T14:34:20.401 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block f261faahjf7tjfi8... #11 @ 2022-08-22T14:34:20.500 signed by leap [trxs: 0, lib: 10, confirmed: 0]
info  2022-08-22T14:34:21.906 nodeos    producer_plugin.cpp:2434      produce_block        ] Produced block ndj573hyd7oedjdj... #12 @ 2022-08-22T14:34:21.000 signed by leap [trxs: 0, lib: 11, confirmed: 0]
...
```

정상적으로 nodeos 가 동작 중이라면 위와 같이 로그에서 블록이 생성되고 있는 과정을 확인할 수 있습니다.

이제 로컬에서 단독으로 실행되는 Antelope 노드 만들어 보았습니다. 일단 여기까지 진행되었으면 개발용 노드가 완성된 것입니다. 만약 nodeos 에서 사용하는 플러그인과 옵션에 대한 상세한 내용이 필요하면 [nodeos Plugin 상세](https://app.gitbook.com/s/YZT0OiBQKuAU7OjJoCgQ/\~/changes/mhCOoiYoXaVjjMMGXHdI/basic-antelope-leap/nodeos-plugin-details) 단원을 참조합니다.
