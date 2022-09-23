# 스마트 컨트랙트 메모리

## Antelope 컨트랙트 데이터 저장소

우리는 스마트 컨트랙트 테이블에 대해 이야기하는 데 익숙하지만 실제로는 이진 트리입니다. 룩업 키는 정렬된 집합에 저장되며 요소를 찾아야 할 때마다 이진 검색을 수행합니다. 이는 또한 범위 내에서 키를 효율적으로 셀 수 없다는 것을 의미합니다. 키를 세려면 각 키를 방문해야 합니다. 데이터 모델을 더 잘 이해하려면 Antelope CDT의 일부인 multi\_index.hpp 안을 살펴보는 것이 유용하며 패키지에서 CDT를 설치한 경우 /usr/opt/eosio.cdt/1.8.1/include/eosolib/contracts/eosio/에서 찾을 수 있습니다. 일반적으로 작동 방식을 이해하려면 CDT와 함께 제공되는 헤더 파일을 살펴보는 것이 좋습니다. 스마트 계약 데이터 상호 작용의 가장 중요한 부분은 네임스페이스 internal\_use\_do\_not\_use라는 재미있는 이름을 가진 장소에서 정의됩니다. 내부에는 Nodeos에 의해 WASM 가상 머신으로 내보낸 여러 내재적 요소 또는 함수가 있습니다. 안타깝게도 헤더에는 이러한 하위 수준의 기능이 설명되지 않으므로 nodeos 소스의 라이브러리/체인/포함/eosio/chain/webassembly/interface.hpp에서 해당 기능을 살펴봐야 합니다. 이러한 함수가 정의하는 것은 사실 nodeos에 의해 관리되는 계약 데이터에 대한 낮은 수준의 액세스입니다. 계약 테이블에 행을 만드는 함수는 어떤 일이 일어나고 있는지 잘 보여줍니다.
