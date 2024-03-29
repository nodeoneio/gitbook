# State History Plugin

## 개요

`state_history_plugin` 을 사용하면 블록체인의 과거 상태 데이터를 가져올 수 있습니다. 이 플러그인은 연결되어 있는 다른 노드로부터 블록체인 데이터를 수신하고 그 데이터를 파일에 캐시해 놓습니다. 또한 클라이언트 애플리케이션이 연결되면 `nodeos` 시작 시 설정한 플러그인 옵션에 따라서 블록체인 데이터를 전송할 수 있도록 소켓을 리스닝 합니다.

## 사용법

다음과 같이 `config.ini` 또는 명령줄에서 사용할 수 있습니다.

```
# config.ini
plugin = eosio::state_history_plugin
[options]

# command-line
nodeos ... --plugin eosio::state_history_plugin [operations] [options]
```

## State History Plugin 명령줄 전용 옵션

아래 명령은 `nodeos` 명령줄에서만 사용할 수 있습니다.

```
--delete-state-history                state history 파일을 클리어한다.
```

## State History Plugin 설정 옵션

다음은 `nodeos` 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

{% code overflow="wrap" %}
```
eosio::state_history_plugin 설정 옵션

--state-history-dir arg (="state-history")
state-history 디렉토리의 위치(절대 경로 또는 data 디렉토리에 대한 상대 경로).

--state-history-retained-dir arg
유지하려는 state-history 디렉토리의 위치(절대 경로 또는 data 디렉토리에 대한 상대 경로).

--state-history-archive-dir arg
아카이빙 하려는 state-history 디렉토리의 위치(절대 경로 또는 data 디렉토리에 대한 상대 경로). 값이 설정되어 있지 않으면 상한값을 넘는 블록 파일은 모두 삭제된다. 이 디렉토리의 파일들은 nodeos 에서 더 이상 접근하지 않기 때문에 사용자가 원하는대로 사용할 수 있다.

--state-history-stride arg
블록 번호가 stride 의 배수가 되면 블록 로그 파일을 분할. stride 값에 도달하면 현재의 히스토리 로그와 인덱스는 *-history-<start num>-<end num>.log/index 로 이름이 변경되며, 최신의 블록 정보를 기반으로 새로운 히스토리 로그와 인덱스가 생성된다. 이 형식으로 저장된 파일들은 히스토리 로그를 생성하고 확장하는데 사용된다.

--max-retained-history-files arg
유지하려는 히스토리 파일 그룹의 최대 개수. 이 파일들을 질의하는데 사용 할 수 있다. 이 숫자에 다다르면 기존 파일은 archive dir 로 옮겨지거나 삭제(archive dir 이 지정되어 있지 않은 경우)된다. 남아 있는 히스토리 로그 파일은 사용자가 건드리면 안 된다.

--trace-history
history 추적을 사용함.

--chain-state-history
chain state history 를 사용함.

--state-history-endpoint arg (=127.0.0.1:8080)
수신되는 커넥션을 리스닝하는 엔드포인트. 주의: 이 포트는 내부 네트워크를 대상으로만 개방할 것.

--state-history-unix-socket-path arg
연결을 리스닝하는 유닉스 소켓을 생성하기 위한 경로. data 디렉토리의 상대경로.

--trace-history-debug-mode
trace history 추적을 위한 디버그 모드를 사용한다.

--state-history-log-retain-blocks arg 
설정하면 arg 의 숫자 만큼의 최근 블록만을 저장하도록 주기적으로 state history 파일을 잘라낸다.
```
{% endcode %}

## 예제

History-tools prune

* [Source code](https://github.com/EOSIO/history-tools/)
* [Documentation](https://eosio.github.io/history-tools/)

## 의존성

* chain\_plugin

### 의존성 로딩 예제

다음과 같이 `chain_plugin` 을 사용하도록 설정되어 있어야 합니다.

```
# config.ini
plugin = eosio::chain_plugin --disable-replay-opts

# command-line
nodeos ... --plugin eosio::chain_plugin --disable-replay-opts
```

## How-To 가이드

[이전 History 없이 빠르게 시작하는 방법](https://www.notion.so/History-49d89c24bdb64fb39e1807a3a278724e)

[How to replay or resync with full history](https://www.notion.so/How-to-replay-or-resync-with-full-history-f402eab880ae41caadd6cbf8c8f9c3b0)

[Full State History를 가진 스냅샷을 작성하는 방법](https://www.notion.so/Full-State-History-abde89028937485ab031a51690a7de04)

[Full State History를 가진 스냅샷을 복구하는 방법](https://www.notion.so/Full-State-History-d384168d326b4fcaa9e311f3499e81c0)
