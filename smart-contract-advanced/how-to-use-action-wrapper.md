# 액션 래퍼 사용법

## 액션래퍼 코드 레퍼런스

액션래퍼 코드는 다음 레퍼런스를 참조합니다.

[https://developers.eos.io/manuals/eosio.cdt/latest/structeosio\_1\_1action\_\_wrapper](https://developers.eos.io/manuals/eosio.cdt/latest/structeosio\_1\_1action\_\_wrapper)

## 시작하기 전에

먼저 다음 사항들이 준비 되었는지 확인합니다.

* [Antelope 개발 환경](../basic-antelope-leap/install-leap-software.md) 준비.
* `multi_index_example.hpp` 파일에 정의된 `multi_index_example` 스마트 컨트랙트.
* user 키를 가지고 정수값 n을 수정할 수 있는 mod 액션.

다음 코드부터 준비하여 시작하겠습니다.

```cpp
class [[eosio::contract]] multi_index_example : public contract {
  // ...
  [[eosio::action]] void mod( name user, uint32_t n );
  // ...
}
```

## 절차

다음은 스마트 컨트랙트에 있는 기존 mod 액션을 래핑하는 mod\_action 액션 래퍼를 만들고 사용하기 위한 순서입니다.

#### 액션 래퍼를 정의

* `eosio::action_wrapper` 템플릿을 사용하여 mod 액션을 래핑하는 액션 래퍼를 정의합니다. 이 메소드는 첫 번째 매개변수로 `eosio::name` 타입의 액션 이름을 받고, 두 번째 매개변수로 액션 메소드의 참조자를 받습니다.

```cpp
class [[eosio::contract]] multi_index_example : public contract {
  // ...
  [[eosio::action]] void mod(name user);
  // ...
+  using mod_action = action_wrapper<"mod"_n, &multi_index_example::mod>;
  // ...
}
```

#### 액션 래퍼 사용

* 헤더 파일을 include 합니다. 액션 래퍼를 사용하려면 액션 래퍼가 정의된 헤더 파일을 include 해야합니다.

```cpp
#include <multi_index_example.hpp>
```

* 액션 래퍼 `mod_action` 을 인스턴스화 합니다. 첫 번째 매개변수로서 액션을 전달하는 컨트랙트를 지정합니다. 여기서 이 컨트랙트는 `multiindexex` 계정으로 배포된다고 가정합니다. 그리고 구조체는 두 개의 매개변수로 정의되는데, 첫 번째는 get\_self() 을 호출하여 얻을 수 있는 자신의 계정이고, 두 번째는 active 권한입니다. 이 두개의 매개변수는 필요에 따라 변경할 수 있습니다.

{% code overflow="wrap" %}
```cpp
#include <multi_index_example.hpp>

+multi_index_example::mod_action mod_action("multiindexex"_n, {get_self(), "active"_n});
```
{% endcode %}

* 액션 래퍼를 사용하여 액션을 전달 \
  액션 래퍼의 send 메소드를 호출하고 mod 액션의 매개변수를 인자로 전달합니다.

```cpp
#include <multi_index_example.hpp>

multi_index_example::mod_action modaction("multiindexex"_n, {get_self(), 1});

+modaction.send("eostutorial"_n, 1);
```

multi\_index 의 전체 예제는 아래를 참조합니다.\
[https://github.com/EOSIO/eosio.cdt/tree/master/examples/multi\_index\_example](https://github.com/EOSIO/eosio.cdt/tree/master/examples/multi\_index\_example)
