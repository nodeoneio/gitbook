---
description: Data design
---

# 데이터 설계

확장 가능하고 많은 데이터를 효율적으로 처리해야 하는 애플리케이션을 구축하려면 적절하게 데이터 설계를 해야 합니다. 특히 Antelope 스마트 컨트랙트 개발자는 여러 가지 한계에 직면할 것이므로 적절한 데이터 설계가 훨씬 더 중요해 집니다.

다음은 스마트 컨트랙트 프로젝트를 설계할 때 일반적으로 고려해야 할 사항들입니다.

* 컨트랙트 데이터에 무엇을 저장해야 하는가. 그리고 블록체인 외부에는 무엇을 저장해야 하는가.
* 컨트랙트는 데이터 요소를 찾기 위해 어떤 질의를 실행해야 하는가. 이러한 질의에 대한 인덱스를 구성해야 한다.
* 블록체인 클라이언트가 필요한 데이터를 찾기 위해 어떤 질의를 실행할 것인가. 이러한 질의에 대한 인덱스를 추가하거나 데이터를 검색하는 오프체인 메소드를 찾아야 할 수도 있다.&#x20;
* 실제로 작동하는 인덱스를 어떻게 구성할 것인가.&#x20;
* 데이터 모델은 완성되었는가. 이 장에서 자세히 설명했듯이, 데이터 구조를 수정하는 것은 비용이 많이 드는 프로세스이다.
