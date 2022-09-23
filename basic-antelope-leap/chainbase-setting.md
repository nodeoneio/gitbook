# 체인베이스 설정

## 체인베이스란?

체인베이스는 블록원의 전 CTO 댄 라리머가 탈중앙화 암호화폐인 비트쉐어 프로젝트에서 근무할 당시의 경험에 기반하여 개발된, 블록체인에 최적화된 데이터베이스 입니다. 체인베이스는 버전컨트롤이 가능하며 트랜잭션을 지원하는 고성능 데이터베이스입니다.

체인베이스는 탈중앙화 소셜 미디어 플랫폼인 Steemit 과 같은 프로젝트에서 메모리 매핑된 파일을 지원하기 위해 업데이트 될 때 채택된 바가 있습니다. 또한 Antelope 의 기본 데이터베이스 이기도 하며 비트셰어와도 통합되어 있습니다.

체인베이스는 `Boost.MultiIIndex` 를 바탕으로 만들어진 경량 인 메모리(In Memory) 데이터베이스입니다. `Boost.MultiIndex` 는 서로 다른 정렬 및 접근 의미(access semantics) 를 가진 하나 이상의 인덱스를 가질 수 있는 컨테이너 타입입니다. 이는 탐색과 정렬을 위한 많은 인덱스를 가질 수 있는 체인베이스 오브젝트 테이블(Chainbase object tables) 을 정의할 수 있는 능력을 체인베이스에 부여합니다.

체인베이스는 성능을 위해 메모리 맵 파일을 사용합니다. 때문에 엄격하게 메모리 위에 올라간 데이터베이스의 크기를 제한한다는 문제점이 있습니다. 일단 데이터베이스 파일의 크기가 시스템 메모리의 크기를 넘어가 버리게 된다면 OS 가 메모리 맵 파일의 데이터를 페이징 하게 되므로 성능 저하가 나타나게 됩니다.

체인베이스는 동시에 쓰기(concurrent writes) 를 지원하지 않습니다. 읽기의 경우 두 개 이상의 스레드가 같은 시간에 수행할 수는 있지만 쓰기의 경우 모든 작업은 반드시 동기화되어야 합니다. 필요하다면 사용자들이 직접 이 기능을 구현해야 합니다.

### 주요 기능

* 높은 성능: 메모리 맵 파일을 과 연동된 인 메모리 데이터를 사용하여 빠르게 실행할 수 있습니다.
* 블록체인 어플리케이션에서 사용하기 위하여 설계되었습니다.
* 다수의 테이블을 지원하며 각 테이블을 검색하고 정렬하기 위한 여러 인덱스를 가질 수 있습니다.
  * 메모리 맵 파일위에서 동작하는 여타 블록체인 데이터베이스들이 있지만, 대부분은 테이블에서 다중 인덱스를 가질 수 없게 되어 있습니다. 체인베이스는 여러가지 키를 사용하여 효율적으로 정렬 및 검색을 수행할 수 있습니다.
* 쉽게 옮길 수 있습니다(semi-portable). 상태(State)는 지속적이며, 다수의 프로세스에서 공유할 수 있지만, 다수의 머신에 걸쳐서 필요한 것은 아닙니다.
  * 데이터베이스 파일은 이를 생성한 머신, 컴파일러, 프로세스의 메모리 레이아웃에 종속적입니다. 따라서 파일을 다른 메모리 레이아웃과 컴파일러를 가진 머신으로 옮기면 예상치 못한 동작을 할 가능성이 높습니다.
* 되돌리기(undo)를 할 수 있는 nested transactional 쓰기를 지원합니다.
* 관계형 데이터베이스와 키-값 데이터베이스를 모두 지원합니다.

## 체인베이스 사용법

nodeos 설정 파일(config.ini) 또는 커맨드라인 상에서 chain plugin 을 사용하도록 지정합니다.

```bash
# config.ini
plugin = eosio::chain_plugin

# command-line
nodeos ... --plugin eosio::chain_plugin[options]
```

체인베이스에 사용되는 chain plugin 은 다음과 같은 설정 옵션을 가지고 있습니다.

### data-dir

* data-dir 은 nodeos 에서 사용하는 옵션으로, nodeos 의 런타임 데이터가 저장되는 디렉토리입니다.
* 현재 nodeos 가 실행되고 있는 경로가 디폴트 data-dir 가 됩니다.
* 체인베이스 데이터는 data-dir 의 하위 디렉토리에 저장됩니다.
* data-dir/state/shared\_memory.bin 이 체인베이스 데이터입니다. 체인베이스 데이터베이스 파일은 비가역성 블록(irreversible blocks)을 가지고 있습니다.
* 다음과 같이 config.ini 또는 커맨드라인에서 지정할 수 있습니다.

```bash
#config.ini
data-dir = ~/nodeos/data

# command line
nodeos ... --data-dir=~/nodeos/data
```

### backing-store

* backing-store 옵션은 nodeos 에게 저장소로 사용할 백엔드 데이터 스토어를 전달할 때 쓰입니다.
* 현재 chainbase 와 rocksdb 를 지원합니다.(\*rocksdb 는 eosio 2.1에서만 지원하며 성능 문제로 Antelope 에서는 지원하지 않습니다.)
*   체인베이스가 디폴트이기 때문에 옵션을 생략해도 되고 아래와 같이 직접 지정해도 됩니다.

    ```bash
    # config.ini
    plugin = eosio::chain_plugin
    backing-store=chainbase

    #command-line
    nodeos ... --plugin eosio::chain_plugin --backing-store=chainbase
    ```

### blocks-dir

* 블록 데이터가 저장될 위치의 경로입니다.
* 디폴트 값은 \<data-dir>/blocks 디렉토리입니다.
* 여기에 blocks.log 파일이 위치한다.
* nodeos 는 이 경로에 있는 blocks.log 파일에 비가역성 블록을 유지합니다.
*   체인베이스의 경우, 이 경로의 하위 디렉토리에 가역성 블록을 포함하는 파일을 저장합니다. 디폴트 경로는 \<blocks-dir>/reversible/shared\_memory.bin 입니다.

    ```bash
    #config.ini
    plugin = eosio::chain_plugin
    blocks-dir=~/nodeos/block

    #command-line
    nodeos ... --plugin eosio::chain_plugin --blocks-dir=~/nodeos/block
    ```

### chain-state-db-size-mb

* 비가역성 블록을 가지는 체인베이스 데이터베이스의 최대 크기(MB)를 지정할 수 있습니다.
* 디폴트 크기는 1024MB 입니다.
* 값이 클수록 체인베이스 데이터베이스가 더 많은 데이터를 가질 수 있으며 작으면 더 적은 데이터가 저장됩니다. 디폴트 값으로도 개인적인 개발 환경을 구축하는데는 문제 없습니다.
* 운영 환경에서는 서버 머신의 실제 메모리 크기와 같거나 그보다 작게 설정합니다.

```bash
#config.ini
plugin = eosio::chain_plugin
chain-state-db-size-mb=500

#command-line
nodeos ... --plugin eosio::chain_plugin --chain-state-db-size-mb=500
```

### chain-state-db-guard-size-mb

* 비가역성 블록이 저장된 체인베이스 데이터베이스 파일의 남은 공간을 얼마만큼 허용할지 상한선을 설정합니다.
* 이 상한선을 넘기게 되면 nodeos 가 안전하게 종료됩니다.
* 디폴트 값은 128MB 입니다.

```bash
#config.ini
plugin = eosio::chain_plugin
chain-state-db-guard-size-mb=50

#command-line
nodeos ... --plugin eosio::chain_plugin --chain-state-db-guard-size-mb=50
```

{% hint style="info" %}
Guard-size 가 필요한 이유

체인베이스 데이터베이스 파일이 허용된 크기를 넘어버리면 비록 순간적 일지라도 데이터가 망가져 버릴 수 있습니다. 따라서 guard-size 는 이러한 시나리오에 대비한 대책으로서 만들어졌습니다. 블록/트랜잭션/연산이 얼마나 많은 데이터를 데이터베이스에 추가하게 될 지 예측할 수 없기 때문에 이 크기는 체인 데이터베이스가 늘어남에 따라 가장 큰 싱글 오퍼레이션보다 큰 사이즈로 지정해야 합니다. 새로운 블록을 추가하는 작업이 주로 발생하는데, 이는 디폴트 크기의 체인 위에 "bulk loading data" 를 허용하기 위한 디폴트 데이터베이스의 1/8 번째로서 선택되었습니다. 블록의 디폴트 제한이 1MB인 경우 이는 원본 데이터를 데이터베이스 데이터로 128배 증폭합니다.
{% endhint %}

### database-map-mode

* mapped, heap, locked 의 3가지 옵션을 제공합니다.
* 디폴트 값은 mapped 입니다.
* mapped: 메모리 매핑된 파일을 나타냅니다.
* heap: swappable 메모리에 미리 로드된 데이터베이스 이며 가능한 많은 페이지를 사용합니다.
* locked: 메모리에 lock 된, 미리 로드된 데이터베이스 이며, 가능한 많은 페이지를 사용합니다.
* heap / locked 는 32비트 버전 윈도우 시스템에서는 사용할 수 없습니다.

```bash
#config.ini
plugin = eosio::chain_plugin
database-map-mode=mapped

#command-line
nodeos ... --plugin eosio::chain_plugin --database-map-mode=mapped
```

* **Mapped vs Heap vs Locked**\
  mapped 는 디폴트 값이며 대부분의 경우 유연하게 사용할 수 있습니다. 커널 설정에 기반하여 디스크에 데이터를 페이징하며, 미리 DB 를 로딩하지 않기 때문에 콜드스타트 시에는 성능이 저하될 수 있습니다. Heap 은 시작할 때 모든 데이터를 가상메모리에 복사합니다. 이에 따라 다음과 같은 두 가지 특징을 가집니다.
  * 시스템을 예열(pre-warming)하기 때문에 콜드스타트 성능이 나아집니다.
  * 셧다운 시 데이터를 기록해야 합니다.
* 셧다운 작업은 오래 걸릴 수 있으며, 그 동안에 변경된 내용은 디스크에 기록되지 않습니다. 성능 면에서는 Locked 가 가장 낫습니다. 이 옵션 또한 모든 데이터베이스를 읽어오지만 물리적인 메모리에 고정(lock) 시키기 때문에 가상 페이징은 발생하지 않습니다. 단, 빈 메모리 공간에 모든 데이터베이스를 적재 할 수 있을만큼 큰 물리 메모리가 필요합니다.
* 거대한 페이지(Huge page) 메모리 맵 파일은 가상 메모리 세그먼트라고 정의할 수 있습니다. 가상메모리는 실제 물리 메모리(RAM)보다 많은 메모리를 가지고 있는 것 처럼 이용할 수 있는 메모리 관리 기술입니다. 어떤 프로세스가 추가적인 RAM 을 필요로 한다면, CPU 는 그것을 덩어리(Chunk)로 RAM 에 할당합니다. 이러한 덩어리를 페이지라고 부르며, 일반적인 크기는 4KB 입니다.\
  RAM이 부족하게 되면, 페이지는 미래 어느 시점에 디스크에 저장되며 그만큼의 여유 메모리 공간을 확보하게 됩니다. CPU는 특정 페이지를 찾아서 RAM으로 불러오기 위해 디스크에 접근해야할 수도 있습니다. 페이지 숫자가 많으면 많을 수록 페이지를 찾아서 메모리에 가져올 때 까지 시간이 많이 걸리게 됩니다.

많은 플랫폼에서 4KB 보다 큰 페이지를 적재할 수 있게 지원합니다. 이렇게 4KB 보다 큰 페이지를 huge page 라고 부릅니다.(몇몇 플랫폼은 large pages 라고도 부릅니다.) Heap 과 Locked 옵션의 경우 플랫폼에서 지원하고 , eosio 가 필요한 전처리기 명령을 사용하여 컴파일 된다면, huge page 를 사용할 수 있습니다. EOSIO 에서는 EOSIO 가 돌아가고 있는 플랫폼에 따라 2MB 또는 1GB 크기의 페이지를 할당할 수 있습니다. 이를 통하여 페이지의 숫자를 줄일 수 있으므로, 어디에 메모리 매핑되어 있는지 찾을때 필요한 시간을 줄일 수 있습니다.

## 체인베이스 모니터링

nodeos 는 블록체인의 분석 데이터를 가져올 수 있는 DB Size API Plugin 이라는 플러그인을 제공합니다. 체인베이스의 전체 크기와 인덱스 당 행(row) 의 수와 함께 사용/미사용 바이트 용량 정보를 보여줍니다. nodeos 문서에 종속 플러그인에서 사용하는 여러가지 옵션들이 설명되어 있으니 이를 참조하여 설정할 수 있습니다.

{% code overflow="wrap" %}
```bash
#config.ini
plugin = eosio::chain_plugin
[options]
plugin = eosio::http_plugin
[options]
plugin = eosio::db_size_api_plugin
[options]

#command-line
nodeos ... --plugin eosio::chain_plugin [options] --plugin eosio::http_plugin [options] --plugin eosio::db_size_api_plugin [options]
```
{% endcode %}

이러한 정보를 가져오려면 nodeos 가 동작중인 서버로 아래와 같은 형식의 HTTP 요청을 보냅니다.

```bash
{protocol}://{host}:{port}/v1/db_size/get
```

이 플러그인을 활용하면, nodeos 가 생성한 체인베이스 데이터베이스상의 guard size 를 지정함과 동시에, nodeos 를 모니터링하고 필요에 따라 정상 종료 시킨 다음 설정을 수정하고 다시 기동시킬 수 있도록 nodeos 를 모니터링하는 스크립트를 작성할 수도 있습니다.

체인베이스와 관련된 모든 문제들은 nodeos 인스턴스를 실행하는데 치명적일 수 있지만 사용자 측에서 대응할 수 있는 문제는 메모리 고갈 뿐입니다. 따라서 지속적으로 db size api 플러그인에서 제공하는 지표들을 모니터링 할 필요가 있습니다. 메모리 고갈이 임박하면 nodeos 를 정상 종료시키고 설정을 변경한 뒤 다시 nodeos를 시작하는것이 좋습니다.
