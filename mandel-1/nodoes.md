# Nodoes 설정 옵션

Nodeos 는 CLI(명령줄 인터페이스) 응용 프로그램입니다. 따라서 명령줄에서 수동으로 시작하거나 자동화된 스크립트를 이용하여 시작할 수 있습니다. Mandel 노드가 어떤 동작을 하는가는 설정한 nodeos 플러그인과 그 플러그인의 옵션에 따라 결정됩니다. nodeos 프로그램의 옵션은 nodeos 전용 옵션과 플러그인 전용 옵션의 두 가지 주요 옵션 카테고리로 나눌 수 있습니다.

Nodeos 플러그인의 옵션은 nodeos 실행 시 명령줄에서 옵션으로 입력하거나 `config.ini` 설정 파일내에 입력할 수 있습니다. Nodeos 데몬 자체에 관한 옵션은 명령줄 에서만 입력할 수 있습니다. nodeos 의 CLI 옵션 및 `config.ini` 옵션 정보는 `nodeos --help` 를 실행하면 확인할 수 있습니다.

## Nodeos 전용 옵션

Nodeos 전용 옵션은 블록체인 데이터가 있는 디렉토리 설정, nodeos 설정 파일 이름 지정, 로깅 설정 파일의 이름과 경로 설정 등 nodeos 프로그램 자체 설정을 위한 용도로 주로 사용됩니다. `nodeos --help` 를 입력하면 도움말이 출력되는데, 다음과 같이 출력된 내용의 하단에서 nodeos 전용 옵션을 확인할 수 있습니다.

```
$ nodeos --help
...
....
...
Application Config Options:
  --plugin arg                          Plugin(s) to enable, may be specified 
                                        multiple times

Application Command Line Options:
  -h [ --help ]                         Print this help message and exit.
  -v [ --version ]                      Print version information.
  --full-version                        Print full version information.
  --print-default-config                Print default configuration template
  -d [ --data-dir ] arg                 Directory containing program runtime 
                                        data
  --config-dir arg                      Directory containing configuration 
                                        files such as config.ini
  -c [ --config ] arg (=config.ini)     Configuration file name relative to 
                                        config-dir
  -l [ --logconf ] arg (=logging.json)  Logging configuration file name/path 
                                        for library users
```

## 플러그인 전용 옵션

플러그인 전용 옵션은 nodeos 플러그인의 동작을 제어합니다. 플러그인 전용 옵션마다 고유한 이름이 있으므로 명령줄이나 `config.ini` 파일 내에서 원하는 방식과 순서대로 기입할 수 있습니다. 일반적으로 어떤 플러그인과 그 플러그인의 옵션들을 한 데 묶어 기입하는 것이 관리하기 편리합니다. 플러그인 전용 옵션을 설정할 때는 반드시 `--plugin` 옵션을 사용하여 해당 플러그인도 함께 사용하도록 설정해야 합니다. 그렇지 않으면 해당 옵션은 무시됩니다.

각 플러그인 전용별 옵션에 대한 자세한 내용은 플러그인 섹션을 참조할 수 있습니다.

대부분의각 `config.ini` 옵션에는 그에 대응하는 CLI 옵션이 존재합니다. 예를 들어 CLI 옵션 `--plugin eosio::chain_api_plugin` 은 `config.ini` 의 `plugin = eosio::chain_api_plugin` 에 대응됩니다.

`config.ini` 에서 사용할 수 없는 CLI 옵션도 일부 있습니다. 예를 들어 `state_history_plugin` 의 `--delete-state-history` 옵션이나 `chain_plugin` 의 `--genesis-json` 과 같은 플러그인 전용 옵션은 `config.ini`에서 사용할 수 없습니다. 이러한 옵션은 nodeos --help 실행한 후 아래 예시와 같이 `Command Line Options for...` 라는 섹션에서 찾아볼 수 있습니다.

```
$ nodeos --help
....
Command Line Options for eosio::chain_plugin:
  --genesis-json arg                    File to read Genesis State from
  --genesis-timestamp arg               override the initial timestamp in the 
                                        Genesis State file
...
```

## **config.ini 설정 파일 위치**

디폴트로 `config.ini` 는  `~/.local/share/eosio/nodeos/config` 폴더에 위치합니다.

직접 작성한 커스텀 `config.ini` 파일을 사용하려면 `nodeos` 실행 시 명령줄에서 `-config <path>/config.ini` 옵션을 설정하면 됩니다.

## Nodeos 실행 예제

다음은 블록 생산자(BP)의 노드를 설정할 때 실행하는 일반적인 nodeos 실행 예제입니다.

```bash
$ nodeos \\
  -e -p mandel \\
  --data-dir /users/mydir/eosio/data     \\
  --config-dir /users/mydir/eosio/config \\
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

* 블록 생성자로 설정하여 블록을 생성하도록 합니다. (`e`)
* 블록 생성자 이름을 "mandel" 로 지정한다. (`p`)
* 블록체인 data 디렉토리를 설정한다. (`-data-dir`)
* `config.ini` 가 위치한 디렉토리를 설정합니다. (`-config-dir`)
* 다음 플러그인들을 로딩합니다. `producer_plugin`, `chain_plugin`, `http_plugin`, `state_history_plugin` (`-plugin`)
* `chain_plugin` 의 전용 옵션들을 설정합니다. (`-contracts-console`, `-disable-replay-opts`)
* `http-plugin`의 전용 옵션들을 설정합니다. (`-access-control-allow-origin`, `-http-validate-host`, `-verbose-http-errors`)
* `state_history_plugin`의 전용 옵션들을 설정합니다. (`-state-history-dir`, `-trace-history`, `-chain-state-history`)
* `stdout` 와 `stderr` 로 출력되는 로그를 `nodeos.log` 파일로 넘깁니다. (`nodeos.log 2>&1`)
* 프로세스를 백그라운드로 돌리고 쉘 프롬프트로 돌아옵니다. (`&`)
