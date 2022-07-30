# CDT(Contract Development Toolkit) 이란

CDT(구 EOSIO.CDT)는 WASM(Web Assembly)용 툴체인으로 EOSIO/Mandel 플랫폼의 스마트 컨트랙트 개발에 사용하는 도구입니다. CDT 는 범용 Web Assembly 툴체인일 뿐만 아니라 EOSIO 스마트 컨트랙트 구축을 지원하기 위해 EOSIO에 최적화되어 있습니다. CDT는 툴체인은 clang7 을 중심으로 구축되었으며, 이는 CDT 가 LLVM에서 가장 최근에 사용 가능한 최적화 및 분석을 보유하고 있다는 의미이지만, WASM 타겟은 여전히 실험적인 것으로 간주되기 때문에 일부 최적화는 불완전하거나 사용할 수 없습니다.

이 문서의 하위 문서는 CDT 1.8 기준으로 작성되었습니다. 만델의 경우 적용되지 않은 기능이 있을 수 있습니다.

또한 CDT 를 지원하기 위한 두 가지 툴이 공개 되어 있었습니다.

{% embed url="https://github.com/eosio/ricardian-template-toolkit" %}
Recardian Template Toolkit
{% endembed %}

{% embed url="https://github.com/eosio/ricardian-spec" %}
Recardian Specification
{% endembed %}

Ricardian Template Toolkit 은 리카르디안 컨트랙트를 작성하는데 사용되는 라이브러리 세트입니다.

Ricardian Specification 은 위의 Template Toolkit 을 위한 도구로, 상세 내용을를 작성하는데 사용됩니다.

단 두 가지 도구 모두 개발된지 3년이 지났고 아직 알파 버전이라는 것을 유념하시기 바랍니다.
