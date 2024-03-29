---
description: CFD
---

# Context-Free Data(CFD)란?

## 개요

블록체인은 변하지 않는다는 특성 덕분에 데이터를 안전하게 저장하는 동시에 데이터의 무결성을 보장할 수 있습니다. 그러나 이러한 이점과는 반대로 블록체인에서 필수적이지 않은 데이터를 제거하는 것은 꽤나 복잡한 일입니다.

이러한 문제를 해결하기 위하여 Antelope 기반 블록체인은 트랜잭션 내에서 컨텍스트 프리 데이터(context-free data, 이하 CFD)라는 특수한 섹션을 포함하고 있습니다. 이름에서 알 수 있듯이, CFD 섹션에 저장된 데이터는 이전의 컨텍스트나 종속성과는 관계 없는 것으로 간주되기 때문에 나중에 제거할 수 있으며, 블록체인의 무결성이 훼손되는 일 없이 안전하게 수행됩니다.

CFD가 제거되어도 블록체인의 보안에 문제가 되지는 않습니다. 전체 유효성 검사 모드로 구성된 노드는 삭제된 트랜잭션 데이터가 있는 블록에서 무결성 위반을 감지할 수 있습니다.

## 개념

CFD의 목표는 블록체인을 사용하는 응용 프로그램이 꼭 필요하지 않은 정보를 트랜잭션 내에 저장할 수 있도록 선택지를 제공하는 것입니다. CFD 의 예는 다음과 같습니다.

* 일시적인 성격이나 특성을 가진 차순위 블록 체인 데이터.
* 트랜잭션 메시지와 관련된 단기적이거나 중요하지 않은 데이터.
* 블록체인에 저장된 온라인 기사에 대한 사용자 의견.

일반적으로 블록체인의 동작과 무결성에 영향을 주지 않는 데이터는 CFD로 저장될 수 있습니다. 또한 이러한 일시적 특성을 활용하여 데이터 사용 및 개인 정보에 관한 지역 법률 및 규정을 준수하는 데도 사용할 수 있습니다.

## 데이터 정리하기

CFD 를 사용하는 블록체인 애플리케이션도 블록체인 무결성에 영향을 미치지 않고 콘텐츠를 제거하기를 원할 수 있습니다. 이것은 정리(Prunning) 라고 불리는 과정을 통해 달성될 수 있습니다. 트랜잭션과 관련된 CFD 를 제거하면 블록체인 애플리케이션에 다음과 같은 더 많은 기능을 제공할 수 있습니다.

* 컨텍스트 또는 상호 종속성이 없는 트랜잭션 데이터를 삭제하는 메커니즘.
* 이러한 컨텍스트 프리 데이터를 제거하면서 블록체인 무결성을 유지하는 방법.

CFD 를 정리하면 신뢰할 수 있는 노드 간에 간략한 블록 유효성 검사(Light block validation)만 허용됩니다. 트랜잭션 서명 검증 및 권한 인증 검사가 수반되는 전체 블록 유효성 검사는 정리가 발생한 블록 및 트랜잭션의 무결성 검사를 위반하지 않고서는 완전히 실현 가능하지 않습니다.

{% hint style="info" %}
프라이빗 블록 체인 정리하기.

프라이빗 Antelope 블록체인은 CFD 를 정리함으로서 상당히 많은 이점을 얻을 수 있습니다. 이러한 제어된 환경이라면 신뢰할 수 있는 노드가 간략한 유효성 검사 모드에서 작동할 수 있습니다. 이를 통해 블록체인 애플리케이션은 이 강력한 기능을 위해 사설 EOSIO 블록체인을 사용할 수 있습니다.
{% endhint %}

데이터 정리 지원

`nodeos` 가 다음 사항을 충족하면 CFD 를 정리 할 수 있습니다.

* 정리된 트랜잭션에서 CFD 가 제거된 비가역 블록을 올바르게 처리합니다.
* 임의의 최종 트랜잭션 내에서 기존 CFD를 효율적으로 삭제합니다.
* 상태 기록 플러그인으로 생성된 CFD가 제거된 트랜잭션 추적을 올바르게 처리합니다.
* 상태 기록 플러그인이 사용한 추적 로그에서 최종 트랜잭션 내에서 기존 CFD를 효율적으로 삭제합니다.
* 적용 가능한 트랜잭션에서 CFD가 제거된 블록의 피어 투 피어 동기화입니다.
* 되돌릴 수 없는 블록 로그 및 상태 기록 플러그인 추적 로그 내의 실제 CFD 가지치기에 대한 도구 지원입니다.

{% hint style="info" %}
데이터 정리 도구

노드 운영자는 [eosio-blocklog](https://developers.eos.io/manuals/eos/latest/utilities/eosio-blocklog) 유틸리티를 사용하여 주어진 트랜잭션 내에서 CFD를 정리할 수 있습니다. 이 도구로 CFD 를 정리하는 방법은 [CFD 정리하는 방법](../antelope-leap-advanced/context-free-data-cfd/how-to-prune-cfd.md) 단원을 참조하시기 바랍니다.
{% endhint %}
