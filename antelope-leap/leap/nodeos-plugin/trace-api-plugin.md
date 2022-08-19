# Trace API Plugin

## 개요

`trace_api_plugin` 은 특정 블록에서 만료된 액션(Retired Action) 및 관련 메타데이터를 검색할 수 있는 소비자 중심의 장기(long-term) API 를 제공합니다. 이 플러그인은 파일 시스템에 직렬화된 블록 추적 데이터를 저장하여 나중에 HTTP RPC 요청을 통해 검색할 수 있도록 합니다. 이 API 에 대한 자세한 내용은 [Trace API Reference](https://developers.eos.io/manuals/eos/latest/nodeos/plugins/trace\_api\_plugin/api-reference/index) 를 참조합니다.

## 목적

블록 탐색기나 거래소 같은 서비스/애플리케이션을 Mandel 블록체인과 통합하는 작업을 하게 되면 스마트 컨트랙트 실행 및 예약된 트랜잭션을 포함하여 블록체인이 처리한 전체 작업 기록이 필요할 수 있습니다. `trace_api_plugin` 을 사용하면 이러한 요구를 충족시킬 수 있습니다. 이 플러그인은 다음과 같은 기능을 제공합니다.

* 만료된 액션(retired action) 및 관련 메타데이터의 스크립트.
* 블록 검색을 위한 소비자 중심의 장기(long-term) API.
* Mandel 노드에서 관리할 수 있는 리소스 commitment 를 제공.

`trace_api_plugin` 의 중요한 목표 중 하나는 노드의 리소스(파일 시스템, 디스크 공간, 사용된 메모리 등)의 유지 관리 방법을 개선하는 것입니다. 이 플러그인은, 이 보다 많은 종류의 구성이 가능한 필터링 및 쿼리 기능을 제공하는 기존 `history_plugin` 이나, 구조 체인 데이터, 액션 데이터 및 상태 델타에 액세스할 수 있는 바이너리 스트리밍 인터페이스를 제공하는`State_History_plugin` 과는 목적이 다릅니다.

## 사용법

다음과 같이 `config.ini`  또는 명령줄에서 사용할 수 있습니다.&#x20;

```
# config.ini
plugin = eosio::trace_api_plugin
[options]

# command-line
nodeos ... --plugin eosio::trace_api_plugin [options]
```

## Trace API Plugin 설정 옵션

다음은 nodeos 명령줄 또는 `config.ini` 파일에 설정할 수 있는 옵션들 입니다.

```
eosio::trace_api_plugin 설정 옵션

  --trace-dir arg (="traces")
    trace 디렉토리의 위치. (절대경로 또는 data 디렉토리의 상대경로)
  --trace-slice-stride arg (=10000)
    trace 데이터의 각 "slice"가 파일 시스템에 포함할 블록의 개수.
  --trace-minimum-irreversible-history-blocks arg (=-1) 
    "slice" 파일을 자동으로 제거하기 전에 검색을 위해 LIB 가 지나도 보관하는 블록 개수.
    (-1 = "slice" 파일의 자동 제거가 해제됨)
  --trace-minimum-uncompressed-irreversible-history-blocks arg (=-1)
    LIB 이전까지 압축되지 않은 블록 개수. 압축된 "slice" 파일에 계속 액세스할 수 있지만 검색 시 
    성능이 저하될 수 있다. (-1 값은 "slice" 파일의 자동 압축이 해제됨을 나타냄.)
  --trace-rpc-abi arg
    trace RPC 응답을 디코딩할 때 사용되는 ABI.
    하나 이상의 ABI가 지정되어 있거나 또는 trace-no-abis 플래그를 사용해야 한다.
    ABI는 <account-name>=<abi-def> 형식의 "Key=Value" 쌍으로 지정된다.
    <abi-def>는 다음과 같다.
      유효한 JSON으로 인코딩 된 ABI를 포함하는 파일의 절대 경로.
      유효한 JSON으로 인코딩 된 ABI를 포함하는 파일의 "data-dir"으로부터의 상대 경로.
  --trace-no-abis
    RPC 응답이 ABI를 사용하지 않을 때 사용.
    trace-rpc-abi 설정이 없을 때 이 옵션을 지정할 경우 실패할 수 있다.
    이 옵션은 trace-rpc-api 와 상호 배타적이다.
```

## 의존성

* chain\_plugin
* http\_plugin

### 의존성 로딩 예제

다음과 같이 `chain_plugin, http_plugin` 을 사용하도록 설정되어 있어야 합니다.

```
# config.ini
plugin = eosio::chain_plugin
[options]
plugin = eosio::http_plugin 
[options]

# command-line
nodeos ... --plugin eosio::chain_plugin [options]  \\
           --plugin eosio::http_plugin [options]
```

## 설정 예제

아래는 특정 Mandel 참조 컨트랙트를을 추적할 때 nodeos 명령줄에 `trace_api_plugin` 을 설정하는 방법에 대한 예시입니다.

```bash
$ nodeos --data-dir data_dir --config-dir config_dir --trace-dir traces_dir
--plugin eosio::trace_api_plugin 
--trace-rpc-abi=eosio=abis/eosio.abi 
--trace-rpc-abi=eosio.token=abis/eosio.token.abi 
--trace-rpc-abi=eosio.msig=abis/eosio.msig.abi 
--trace-rpc-abi=eosio.wrap=abis/eosio.wrap.abi
```

## 정의

이 단원에서는 slices, trace log contents, clog format 이 무엇인지 알아보겠습니다. 이 개념들을 숙지하면 `trace_api_plugin` 옵션을 효과적으로 사용하는데 도움이 될 것입니다.

### **Slices**

`trace_api_plugin` 컨텍스트에서 slice 는 지정된 시작 블록을 포함하는 높이와 지정된 끝 블록을 포함하는 높이 사이의 모든 관련된 trace 데이터의 모음으로 정의할 수 있습니다. 예를 들어 0에서 10,000 사이의 slice 는 블록 번호가 0보다 크거나 같고 10,000 보다 작은 모든 블록의 집합입니다. trace 디렉토리에는 slice 모음이 포함되어 있습니다. 각 slice는 trace data 로그 파일과 trace index metadata 로그 파일로 구성됩니다.

* `trace_<S>-<E>.log`
* `trace_index_<S>-<E>.log`

여기서 \<S>와 \<E>는 0으로 채워져 있는, 패딩된 슬라이스의 시작 및 끝 블록 번호입니다. 예를 들어 시작 블록이 5이고, 마지막 블록이 15이며, 진행 속도가 10이면 그 결과는 \<S>가 0000000005이고, \<E>는 0000000015 가 됩니다.

### trace\_\<S>-\<E>.log

trace 데이터 로그는 실제 이진 직렬화된 블록 데이터를 저장하는 추가 전용(append only) 로그입니다. 그 내용에는 액션별 ABI 로 증강된 RPC 요청을 처리하는 데 필요한 트랜잭션 및 액션 추적 데이터가 포함됩니다. 블록 유형은 아래 두 가지를 지원합니다.

* block\_trace\_v0
* block\_trace\_v1

데이터 로그는 로그에 저장된 데이터에 대한 버전 정보를 포함하는 기본 헤더에서부터 시작합니다. `block_trace_v0` 에는 블록 ID, 블록 번호, 이전 블록 ID, 프로덕션 타임스탬프, 블록에 서명한 BP 및 실제 추적 데이터가 포함됩니다.

`block_suffer_v1` 은 트랜잭션 목록과 블록에 포함된 액션 목록뿐만 아니라 genesis 이후의 생산 공정 수(Production Schedule Count)에도 merkle root hash 를 추가합니다.

로그에는 체인의 정상적인 작동의 일부로서 블록체인에서 포크아웃(forked out) 된 블록이 포함될 수 있습니다. 파일의 다음 항목은 항상 이전 항목보다 1이 높거나 포킹으로 인해 같은 번호 또는 그 이하인 블록 번호를 가지게 됩니다. 모든 추적 항목에는 추적 인덱스에 해당하는 slice 파일에 해당 항목이 있습니다. `read-mode=irreversible` 로 설정한노드를 실행하면 fork 된 블록을 방지할 수 있습니다.

### trace\_index\_\<S>-\<E>.log

추적 인덱스 로그 또는 메타데이터 로그는 일련의 이진 시퀀스를 저장하는 추가 전용(append only) 로그입니다. 현재 지원되는 유형은 아래 두 가지가 있습니다.

* block\_entry\_v0
* lib\_entry\_v0

인덱스 로그는 로그에 저장된 데이터에 대한 버전 정보를 포함하는 기본 헤더부터 시작합니다.&#x20;

`block_entry_v0` 에는 데이터 로그 내에서 해당 블록 위치에 대한 오프셋이 있는 블록 ID와 블록 번호가 포함됩니다. 이 항목은 `block_trace_v0` 및 `block_trace_v1` 블록의 오프셋을 모두 찾는 데 사용됩니다.

`lib_entry_v0` 에는 알려진 최신 LIB에 대한 항목이 포함되어 있습니다. reader 모듈은 LIB 정보를 사용하여 사용자에게 irreversible status 를 보고합니다.

### Clog format

추적 로그 파일은 압축 파일이며 확장자는 .clog 입니다.([로그 파일 압축](https://developers.eos.io/manuals/eos/latest/nodeos/plugins/trace\_api\_plugin/index/#compression-of-log-files) 참조). clog은 탐색 가능한 압축 해제 포인트의 인덱스가 끝에 추가된 일반적인 압축 파일입니다. clog 형식 레이아웃은 다음과 같습니다.

![](<../../../.gitbook/assets/image (1) (1).png>)

데이터는 raw zlib 형식으로 압축되며, 일정한 간격으로 full-flush 탐색 지점을 배치합니다. 압축 해제기는 이전 데이터를 읽지 않고도 이러한 탐색 지점부터 시작할 수 있으며 데이터 내에 표시되는 탐색 지점을 문제 없이 통과할 수도 있습니다.

{% hint style="info" %}
**추적 로그 크기 줄이기** \
데이터 압축을 통하여 추적 로그의 증가하는 공간을 20배 가량 줄일 수 있습니다. 예를 들어, EOS 공용 네트워크에서 512개의 검색 지점을 사용하고 테스트 데이터셋을 사용하면 데이터 압축은 전체 데이터에 대해 추적 디렉토리의 증가량을 일일 최대 50GiB 에서 최대 2.5GiB 로 줄일 수 있습니다. 추적 로그 내용의 중복성이 높기 때문에 압축률은 gzip -9 와 비슷합니다. 압축 해제된 데이터도 서비스 저하 없이 [Trace RPC API](https://developers.eos.io/manuals/eos/latest/nodeos/plugins/trace\_api\_plugin/api-reference/index)를 통해 즉시 사용할 수 있습니다.
{% endhint %}

#### 탐색 지점(Seek point)의 역할

파일이 압축될 때 탐색 지점 인덱스는 원래 압축되지 않은 오프셋을 새 압축된 오프셋과 함께 기록하여 매핑을 생성하므로 원래 인덱스 값(압축되지 않은 오프셋)이 압축되지 않은 오프셋보다 가장 가까운 탐색 지점에 매핑될 수 있습니다. 이렇게 하면 스트림 뒷부분에 나타나는 압축되지 않은 파일의 일부에 대한 검색 시간이 크게 줄어들게 됩니다.

## 자동 유지 관리

`trace_api_plugin` 의 주요 설계 목표 중 하나는 파일 시스템 리소스의 수동 관리를 최소화하는 것입니다. 이를 위해 이 플러그인은 추적 로그 파일의 자동 제거와 데이터 압축을 통한 디스크 설치 공간 자동 축소를 용이하게 합니다.

### 로그 파일 제거

`trace_api_plugin` 에서 생성된 이전 추적 로그 파일을 제거하려면 다음 옵션을 사용할 수 있습니다.

```
-trace-minimum-irreversible-history-blocks N (=-1)
```

인수 N이 0 이상이면 플러그인은 현재 LIB 블록 이전의, 디스크상에 있는 N개의 블록만 유지합니다. 블록 번호가 이전 N개 블록보다 작은 추적 로그 파일은 자동으로 제거되도록 예약됩니다.

### 로그 파일 압축

trace\_api\_plugin은 또한 추적 로그 파일에 데이터 압축을 적용하여 디스크 공간을 최적화하는 옵션을 지원합니다.

```
--trace-minimum-uncompressed-irreversible-history-blocks N (=-1)
```

인수 N이 0 이상이면 플러그인은 자동으로 백그라운드 스레드를 설정하여 추적 로그 파일의 비가역 섹션을 압축합니다. 현재 LIB 블록을 통과한 이전 N개의 비가역성 블록은 압축되지 않은 상태로 유지됩니다.

{% hint style="info" %}
추적 API 유틸리티 추적 로그 파일은 [trace\_api\_util](https://developers.eos.io/manuals/eos/latest/utilities/trace\_api\_util) 유틸리티를 사용하여 수동으로 압축할 수도 있습니다.
{% endhint %}

`trace-minimum-irreversible-history-blocks` 및 `trace-minimum-uncompressed-irreversible-history-blocks` 옵션을 통해 리소스 사용을 효과적으로 관리할 수 없는 경우 정기적인 수동 유지 관리가 필요할 수도 있습니다. 이 경우 사용자는 외부 시스템 또는 반복 프로세스를 통해 리소스를 관리하도록 선택할 수 있습니다.

## 수동 유지 관리

`trace-dir` 옵션은 `trace_api_plugin` 에 의해 추적 로그 파일이 저장되는 파일 시스템의 디렉토리를 정의합니다. 이러한 파일은 지정된 slice 를 지나 LIB 블록이 진행되면 안정적이므로 언제든지 삭제하여 파일 시스템 공간을 회수할 수 있습니다. 배포된 Mandel 시스템은 해당 파일이 나타내는 데이터 또는 액세스 중인 nodeos 인스턴스가 있는지 여부에 관계없이 이 디렉토리에서 이러한 파일의 일부 또는 전부를 제거하는 프로세스 외 관리 시스템을 허용합니다.

명목상으로는 사용 가능하지만 수동 유지 관리로 인해 더 이상 사용할 수 없는 데이터는 해당 API 엔드포인트에서 HTTP 404 응답을 받게 됩니다.

{% hint style="info" %}
노드 연산자의 경우 노드 운영자는 외부 파일 시스템 리소스 관리자와 함께 `trace-api-plugin` 및 `trace-minimum-irreversible-history-blocks,trace-minimum-uncompressed-irreversible-history-blocks` 옵션을 통해 노드에서 사용 가능한 기록 데이터(historical data)의 수명을 완전히 제어할 수 있습니다.
{% endhint %}

## API Reference

[https://developers.eos.io/manuals/eos/latest/nodeos/plugins/trace\_api\_plugin/api-reference/index](https://developers.eos.io/manuals/eos/latest/nodeos/plugins/trace\_api\_plugin/api-reference/index)
