# Nodeos Plugin 옵션 상세

## 개요

`nodeos` 는 블록체인 네트워크의 핵심 기능이 구현되어 있으며, 모듈화 된 소프트웨어이기 때문에 플러그인을 설정하여 확장된 기능들을 사용할 수 있습니다. nodeos 는 여러가지 플러그인을 가지고 있지만, 이들 중 nodeos 의 모듈 디자인을 반영하고 있는 `chain_plugin`, `net_plugin`, `producer_plugin`와 같은 플러그인은 거의 필수로 설정해야 합니다. 다른 플러그인들도 유용한 기능을 제공하고 있지만 노드 운영에서 필수는 아닙니다.

{% hint style="info" %}
일반적 런타임 플러그인과는 달리, nodeos 의 플러그인은 컴파일 타임에 빌드됩니다.
{% endhint %}

{% hint style="info" %}
본 문서는 Leap 3.1.0 버전을 기준으로 작성되었습니다. 빠르게 발전하는 Leap 소프트웨어의 특성상 수시로 변경이 있을 수 있으니, 본 문서에서 아직 다루지 않은 내용은 Antelope 공식 깃허브나`help` 명령어로  확인하여 주시기 바랍니다.
{% endhint %}

Leap 는 다음과 같은 플러그인을 제공하고 있으며 상세한 내용은 각각의 단원에서 알아보겠습니다.

* Chain Plugin
* Net Plugin
* Producer Plugin
* HTTP Plugin
* HTTP Client Plugin
* State History Plugin
* Resource Monitor Plugin
* Trace API Plugin
* Txn Test Gen Plugin
* Login Plugin
