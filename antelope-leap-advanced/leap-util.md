---
description: Leap-util utility
---

# Leap-util 유틸리티

## Leap-util 에 대하여

잠시 `nodeos` 이야기를 해 보겠습니다. `nodeos` 는 원래 데몬(daemon)으로 실행되도록 설계된 프로그램입니다.&#x20;

그러나 시간이 지남에 따라 `Leap` 노드 운영자에게 필요한 이런 저런 다양한 기능들이 `nodeos` 에 추가되었습니다. 이렇게 추가된 기능을 사용하면 `nodeos` 는 일반 프로그램과 마찬가지로 실행된 다음, 작업이 끝난 후 바로 종료됩니다.

문제는 `nodeos` 는 `cleos` 와 같은 CLI 도구가 아닌 데몬으로 사용하도록 설계되었기 때문에, `nodeos` 프로그램 내에서 이러한 기능을 호출하는 인터페이스는 `cleos` 에서 사용하는 계층적 하위 명령 인터페이스(hierarchical sub-command interface) 에 비해 세련되지 못한 방식으로 작성되었습니다. 그렇다고 해도 이러한 기능들은 `Leap` 노드 운영자들이 필요로 하기 때문에 만들어진 것이며, 어쨌거나 `nodeos` 실행 파일과 밀접하게 연관되어 있는 것도 사실입니다.

그럼 `cleos` 는 어떨까요? `cleos` 는 사용자가 퍼블릭 API 인터페이스를 사용하여 노드 서버와 통신할 때 필요한 프로그램이지만, `nodeos` 노드 데몬을 운영하지 않는 일반적 사용자를 위한 도구이기도 합니다. 따라서 이런저런 모든 기능들을 `cleos` 에 전부 쑤셔넣는 것은 프로그램의 인터페이스를 난잡하게 하고 `cleos` 사용자들을 혼란스럽게 할 가능성이 높습니다.

그래서 그동안 `Leap` 소프트웨어 패키지에서 `eosio-blocklog` 와 같은 추가 프로그램을 함께 제공하였으며, `nodeos` 를 따로 실행하지 않고도 이 프로그램으로 `Leap` 노드 운영자들에게 유용한 기능들을 사용할 수 있었습니다.

Leap 3.2 버전부터는 `leap-util` 이라는 새로운 프로그램을 도입하였습니다. `leap-util` 프로그램은 특히 `Leap` 노드 운영자에게 도움이 되는 기능들을 탑재하고 있으며, 원래 `nodeos`가 데몬으로 사용하기 위하여 만들어졌다는 점을 고려할 때, `nodeos` 에 포함시키기에 적절하지 않은 모든 기능을 모아두기 위한 목적으로 만든 통합 도구입니다.

또한 `leap-util` 은 `cleos` 사용자에게 친숙한 계층적 하위 명령 인터페이스를 사용합니다.

`leap-util` 는 현재 `eosio-blocklog` 프로그램의 코드를 그대로 가져왔기 때문에 `eosio-blocklog` 에서 제공하는 모든 기능을 포함하고 있으며 시간이 지남에 따라 추가적인 기능, 예를 들어 `nodeos` 의 성격과 다르지만 현재 `nodeos` 에 포함되어 있는 기능과, 이전 `EOSIO` 나 `Leap` 릴리스 에는 존재하지 않았던 완전히 새로운 기능들도 포함될 것입니다.

`eosio-blocklog` 프로그램은 leap 4.0 부터 제거될 것이며, 대신 `leap-util` 의 `block-log` 명령을 사용할 수 있습니다.

Leap 4.0 현재, `leap-util` 은 MVP(Minimum Value Product, 최소 기능 제품) 버전이며 자동 완성을 지원하는 CLI11 명령줄 구문 분석 라이브러리와 더불어 다음과 같은 하위 명령(Subcommand) 들을 제공하고 있습니다.

## Leap-util 명령 상세

{% code overflow="wrap" %}
```
version:    버전 정보 조회
└ client:   클라이언트의 기본 버전 정보 조회
└ full:     클라이언트의 풀 버전 정보 조회

block-log:        Blocklog 유틸리티 명령.
└ print-log:      blocks.log 를 JSON 형태로 출력.
└ make-index:     blocks.log 로 부터 blocks.index 를 생성. 
    └ --blocks-dir TEXT:       blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).
    └ -o, --output-file TEXT:  결과물을 출력할 파일 이름과 경로(상대경로 또는 절대경로). 지정하지 않으면 stdout 으로 출력된다.
    
└ trim-blocklog:        blocks.log 와 blocks.index 을 잘라낸다.
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로)
    └ -f, --first UINT REQUIRED:    유지하려는 첫번째 블록 번호.
    └ -l, --last UINT REQUIRED:     유지하려는 마지막 블록 번호.

└ extract-blocks:        blocks.log 에서 지정된 범위의 블록을 추출하여 output-dir 에 작성한다.
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).
    └ -f, --first UINT REQUIRED:        추출하려는 첫번째 블록 번호.
    └ -l, --last UINT REQUIRED:         추출하려는 마지막 블록 번호.
    └ --output-dir TEXT:        blocks-dir 에서 추출된 블록로그를 출력할 디렉토리.

└ split-blocks:        스트라이드(stride)를 기준으로 blocks.log 를 분할하여 지정된 output-dir 에 출력한다.
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).
    └ --output-dir TEXT:        분할된 블록로그를 출력할 디렉토리.
    └ --stride UINT REQUIRED:        각각의 파일로 분할 출력할 블록의 개수

└ merge-blocks:        'blocks-\d+-\d+.[log,index]' 의 패턴을 가진 블록 로그 파일들을 병합하여 output-dir 디렉토리에 출력. 'blocks-dir' 내의 파일들은 변경되지 않음.
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).
    └ -o, --output-dir TEXT:        병합된 블록로그를 출력할 디렉토리.

└ smoke-test:        blocks.log 와 blocks.index 가 제대로 형성되었고 서로 연관지어져 있는지 빠르게 테스트.
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).

└ vacuum:        제거된(pruned) blocks.log 를 제거되지 않은(un-pruned) blocks.log 에 삽입.
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).

└ genesis:        blocks.log 에서 genesis_state 를 JSON 형태로 추출
    └ --blocks-dir TEXT:        blocks 디렉토리의 위치(절대경로 또는 현 위치에서의 상대경로).
    └ -o, --output-file TEXT:   결과물을 출력할 파일 이름과 경로(상대경로 또는 절대경로). 지정하지 않으면 stdout 으로 출력된다.

snapshot:    스냅샷 유틸리티
└ to-json:    스냅샷 파일을 JSON 형태로 변환.
    └ -i, --intput-file TEXT REQUIRED:    JSON 으로 변환할 스냅샷 파일. Output 파일이 지정되지 않으면 <file>.json 형태로 출력된다.
    └ -o, --output-file TEXT:    결과물을 출력할 파일 이름과 경로(상대경로 또는 절대경로). 지정하지 않으면 <input-file>.json 형태로 출력된다.
    └ --chain-id TEXT:    스냅샷에 체인 ID가 포함되어 있지 않은 경우나 기존 ID 를 덮어씌우고 싶을 때 지정.
    └ --db-size UINT [65536]:    체인 상태 데이터베이스(Chain State Database)의 최대 크기(MiB) 

chain-state:    체인 유틸리티
└ build-info:    빌드 환경 정보를 JSON 형태로 추출
    └ -o, --output-file TEXT:   결과물을 출력할 파일 이름과 경로(상대경로 또는 절대경로).
    └ -p, –print:    콘솔에 출력.
```
{% endcode %}
