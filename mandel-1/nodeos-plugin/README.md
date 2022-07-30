# Nodeos Plugin 상세

### 개요

Nodeos 에는 블록체인 네트워크의 핵심 기능이 구현되어 있으며 모듈화 된 소프트웨어 입니다. Nodeos 의 확장된 기능들을 사용하려면 플러그인을 설정합니다. Nodeos 는 여러가지 플러그인을 가지고 있지만, 이들 중 nodeos 의 모듈 디자인을 반영하고 있는 `chain_plugin`, `net_plugin`, `producer_plugin`와 같은 플러그인은 거의 필수로 설정해야 합니다. 다른 플러그인들도 유용한 기능을 제공하고 있지만 노드 운영에서 필수는 아닙니다.

{% hint style="info" %}
일반적 런타임 플러그인과는 달리, nodeos 의 플러그인은 컴파일 타임에 빌드됩니다.
{% endhint %}

{% hint style="info" %}
본 문서는 Mandel 3.1.0-rc2 버전을 기준으로 작성되었습니다. 빠르게 발전하는 Mandel 소프트웨어의 특성상 많은 기
{% endhint %}
