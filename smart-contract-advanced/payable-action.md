---
description: 스마트 컨트랙트를 호출할 때 비용을 지불하는 방법
---

# 지불 가능한 액션(Payable Action)

이 단원에서는 지불 가능한 액션을 가지는 스마트 컨트랙트를 만드는 방법을 알아보겠습니다.

## 개요

지불 가능한 액션은 스마트 컨트랙트의 어떤 기능을 사용하기 전에, 얼마간의 토큰을 액션으로 전송할 필요가 있을 때 사용하는 액션입니다. 그리고 Antelope asset 타입을 본편에서 다룰 것입니다.

이번에는 특정한 토큰을 받아들이지만, 일정 기간 동안은 토큰이 인출되지 않도록 하는 스마트 컨트랙트 코드를 작성해보겠습니다.

## HODL 토큰

먼저 `eosio:coontract` 을 확장하는 hodl C++ 표준 클래스를 만듭니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract("hodl")]] hodl : public eosio::contract{
  private:
  public:
}
```

이 컨트랙트에 다음과 같은 몇 가지 조건을 설정해야 합니다.

* 이 컨트랙트에서 사용하는 기호/토큰은 무엇인가?
* hodl 은 언제 끝낼 것인가.

이제 이러한 조건을 상수로 정의하겠습니다.

* hodl\_symbol: 이 컨트랙트에서 사용하는 토큰 기호. 본 예제에서는 "SYS" 기호를 사용하겠습니다.
* the\_party: 2022년 2월 22일 화요일 오후 10:22:22에 해당하는 시간값으로, 이 때 Hodl을 종료하도록 설정하겠습니다.

```cpp
#include <eosiolib/eosio.hpp>

using namespace eosio;

class [[eosio::contract("hodl")]] hodl : public eosio::contract {
  private:
    static const uint32_t the_party = 1645525342;
    const symbol hodl_symbol;
  public:

}
```

다음으로 hodl 컨트랙트가 받은 토큰의 수를 추적하는 테이블을 정의합니다.

```cpp
struct [[eosio::table]] balance
{
  eosio::asset funds;
  uint64_t primary_key() const { return funds.symbol.raw(); }
};
```

이 다중 인덱스 테이블 선언할 때 asset 이라는 새 타입을 사용할 것입니다. asset 은 디지털 토큰 자산을 나타내도록 설계된 타입입니다. 자세한 내용은 아래의 asset 참조 설명서를 확인합니다.

[https://developers.eos.io/manuals/eosio.cdt/v1.8/structeosio\_1\_1asset](https://developers.eos.io/manuals/eosio.cdt/v1.8/structeosio\_1\_1asset)

asset 인스턴스의 symbol 멤버가 Primary Key 로 사용됩니다. 여기서 raw() 함수를 호출하면 symbol 변수를 Primary Key 로 사용할 수 있도록 부호 없는 정수로 변환됩니다.

### 생성자

생성자는 hodl\_symbol 을 "SYS"로 초기화 합니다. SYS 는 [토큰 배포, 발급 및 전송](../smart-contracts-dev-environment/contract-dev-workflow/) 단원에서 만들었던 토큰입니다.

{% code overflow="wrap" %}
```cpp
public:
  using contract::contract;
  hodl(name receiver, name code, datastream<const char *> ds):contract(receiver, code, ds), hodl_symbol("SYS", 4){}
```
{% endcode %}

### 현재 시각 (UTC) 가져오기

코드에서 UTC 타임존 시간을 가져오기 위해서 현재의 UTC 시간에 쉽게 접근할 수 있는 함수를 만듭니다.

```cpp
uint32_t now() {
  return current_time_point().sec_since_epoch();
}
```

이제 컨트랙트의 액션을 작성하겠습니다.

### 예금(Deposit)

전송을 하려면 예금 액션이 있어야 합니다.

{% code overflow="wrap" %}
```cpp
[[eosio::on_notify("eosio.token::transfer")]]
void deposit(name hodler, name to, eosio::asset quantity, std::string memo)
{
  if (to != get_self() || hodler == get_self())
  {
    print("These are not the droids you are looking for.");
    return;
  }

  check(now() < the_party, "You're way late");
  check(quantity.amount > 0, "When pigs fly");
  check(quantity.symbol == hodl_symbol, "These are not the droids you are looking for.");

  balance_table balance(get_self(), hodler.value);
  auto hodl_it = balance.find(hodl_symbol.raw());

  if (hodl_it != balance.end())
    balance.modify(hodl_it, get_self(), [&](auto &row) {
      row.funds += quantity;
    });
  else
    balance.emplace(get_self(), [&](auto &row) {
      row.funds = quantity;
    });
}
```
{% endcode %}

지금까지 잘 따라왔다면 이 액션의 코드를 이해하기 어렵지 않을 것입니다.

먼저, 액션은 컨트랙트가 자기 스스로에게 이체하고 있지는 않은지 확인합니다.

```cpp
if (to != get_self() || hodler == get_self()) {
  print("These are not the droids you are looking for.");
  return;
}
```

컨트랙트 계좌로 이체하는 것은 해당 계좌가 eosio.token 컨트랙트가 가지고 있는 것보다 더 많은 토큰을 가지게 되는 잘못된 상황을 만들 수 있기 때문에 위와 같이 확인 할 필요가 있습니다.

그런 다음 이 액션은 다음과 같은 몇 가지 조건을 추가로 확인합니다.

* 인출 시간이 지나지는 않았는가.
* 유효한 수의 토큰이 전송되어 들어오고 있는가.
* 전송되어 들어오는 토큰은 생성자에서 지정한 토큰과 일치하는가.

{% code overflow="wrap" %}
```cpp
check(now() < the_party, "You're way late");
check(quantity.amount > 0, "When pigs fly");
check(quantity.symbol == hodl_symbol, "These are not the droids you are looking for.");
```
{% endcode %}

만약 모든 조건을 통과한다면 액션은 알맞게 잔고를 업데이트 합니다.

```cpp
balance_table balance(get_self(), hodler.value);
auto hodl_it = balance.find(hodl_symbol.raw());

if (hodl_it != balance.end())
  balance.modify(hodl_it, get_self(), [&](auto &row) {
    row.funds += quantity;
  });
else
  balance.emplace(get_self(), [&](auto &row) {
    row.funds = quantity;
  });
```

여기서 중요한 포인트는 deposit 함수가 실제로는 `eosio.token` 컨트랙트에 의하여 실행된다는 것입니다. 이 동작을 이해하려면 `on_notify` 속성을 이해해야 합니다.

### on\_notify 속성

```cpp
[[eosio::on_notify("eosio.token::transfer")]]
```

on\_notify 는 CDT의 속성 중 하나로 스마트 컨트랙트 액션에 달 수 있는 어노테이션 입니다.

어떤 액션에 on\_notify 속성의 어노테이션을 달면 정해진 컨트랙트의 액션에서 알림이 발송된 경우에만 어노테이션이 달린 액션으로 수신 알람이 전달됩니다.

이 예제의 경우 on\_notify 속성은 eosio.token 컨트랙트와과 eosio.token 의 transfer 액션에서 알림이 수신되는 경우에만 수신 알림이 입금 액션에 전달되도록 하고 있습니다.

전송 통지가 hodl deposit 액션에 도달하기 전에 eosio.token 컨트랙트가이 검사를 마쳤을 것이기 때문에 호들러가 실제로 청구한 적절한 양의 토큰을 가지고 있는지 확인할 필요가 없습니다.

### 파티!

Party 액션은 설정된 the\_party 시간이 경과한 후에만 출금을 허용합니다. party 액션은 다음 조건을 가진 deposit 액션과 비슷한 생성자를 가집니다.

* 출금계좌가 처음에 입금을 했던 계좌인지 확인
* 잠겨 있는 잔고를 찾음
* hodl 컨트랙트로 부터 계좌로 토큰을 이체

{% code overflow="wrap" %}
```cpp
[[eosio::action]]
void party(name hodler)
{
  //Check the authority of hodler
  require_auth(hodler);

  //Check the current time has passed the the_party time
  check(now() > the_party, "Hold your horses");

  balance_table balance(get_self(), hodler.value);
  auto hodl_it = balance.find(hodl_symbol.raw());

  //Make sure the holder is in the table
  check(hodl_it != balance.end(), "You're not allowed to party");

  action{
    permission_level{get_self(), "active"_n},
    "eosio.token"_n,
    "transfer"_n,
    std::make_tuple(get_self(), hodler, hodl_it->funds, std::string("Party! Your hodl is free."))
  }.send();

  balance.erase(hodl_it);
}
```
{% endcode %}

완성된 코드는 다음과 같습니다.

{% code overflow="wrap" %}
```cpp
#include <eosio/eosio.hpp>
#include <eosio/print.hpp>
#include <eosio/asset.hpp>
#include <eosio/system.hpp>

using namespace eosio;

class [[eosio::contract("hodl")]] hodl : public eosio::contract {
  private:
    static const uint32_t the_party = 1645525342;
    const symbol hodl_symbol;

    struct [[eosio::table]] balance
    {
      eosio::asset funds;
      uint64_t primary_key() const { return funds.symbol.raw(); }
    };

    using balance_table = eosio::multi_index<"balance"_n, balance>;

    uint32_t now() {
      return current_time_point().sec_since_epoch();
    }

  public:
    using contract::contract;

    hodl(name receiver, name code, datastream<const char *> ds) : contract(receiver, code, ds),hodl_symbol("SYS", 4){}

    [[eosio::on_notify("eosio.token::transfer")]]
    void deposit(name hodler, name to, eosio::asset quantity, std::string memo) {
      if (hodler == get_self() || to != get_self())
      {
        return;
      }

      check(now() < the_party, "You're way late");
      check(quantity.amount > 0, "When pigs fly");
      check(quantity.symbol == hodl_symbol, "These are not the droids you are looking for.");

      balance_table balance(get_self(), hodler.value);
      auto hodl_it = balance.find(hodl_symbol.raw());

      if (hodl_it != balance.end())
        balance.modify(hodl_it, get_self(), [&](auto &row) {
          row.funds += quantity;
        });
      else
        balance.emplace(get_self(), [&](auto &row) {
          row.funds = quantity;
        });
    }

    [[eosio::action]]
    void party(name hodler)
    {
      //Check the authority of hodlder
      require_auth(hodler);

      // //Check the current time has pass the the party time
      check(now() > the_party, "Hold your horses");

      balance_table balance(get_self(), hodler.value);
      auto hodl_it = balance.find(hodl_symbol.raw());

      // //Make sure the holder is in the table
      check(hodl_it != balance.end(), "You're not allowed to party");

      action{
        permission_level{get_self(), "active"_n},
        "eosio.token"_n,
        "transfer"_n,
        std::make_tuple(get_self(), hodler, hodl_it->funds, std::string("Party! Your hodl is free."))
      }.send();

      balance.erase(hodl_it);
    }
};
```
{% endcode %}

이제 위 코드를 배포할 수 있습니다.

### deposit 테스트

먼저 계정을 만들고 배포해 보겠습니다.

{% code overflow="wrap" %}
```cpp
cleos create account eosio hodl EOS7kG7snRRzY9oRBXmYVVitWx1mtp7sUBeZzoXYU9TQWFLRk6As6
cdt-cpp hodl.cpp -o hodl.wasm
cleos set contract hodl ./ -p hodl@active
```
{% endcode %}

앞서 언급한대로 한 대로 컨트랙트는 `eosio.code` 권한이 필요합니다.

```cpp
cleos set account permission hodl active --add-code
```

다음, 테스트 계정을 만들겠습니다.

```cpp
cleos create account eosio han EOS7kG7snRRzY9oRBXmYVVitWx1mtp7sUBeZzoXYU9TQWFLRk6As6
```

앞서 만든 SYS 토큰을 han 에게 전송합니다.

```cpp
cleos push action eosio.token transfer '[ "alice", "han", "100.0000 SYS", "Slecht geld verdrijft goed" ]' -p alice@active
```

마지막으로 SYS 토큰을 han 의 계정에서 hodl 컨트랙트로 전송해 보겠습니다.

```cpp
cleos transfer han hodl '0.0001 SYS' 'Hodl!' -p han@active
```

### 인출 테스트

인출 기능을 테스트하려면, the\_party 변수를 업데이트 해야 합니다. 인출 기능을 테스트할 수 있도록 the\_party 변수를 과거의 어느 시점으로 업데이트 합니다.

```cpp
CONTRACT hodl : public eosio::contract {
  private:
    // 9 June 2018 01:00:00
    static const uint32_t the_party = 1528549200;
```

성공적으로 자금을 인출한 결과는 다음과 같습니다.

{% code overflow="wrap" %}
```cpp
cleos push action hodl party '["han"]' -p han@active

executed transaction: 62b1e6848c8c5e6458b9a0f7600e65574eaf60445be114d224adccc5a962a09a  104 bytes  383 us
#          hodl <= hodl::party                  {"hodler":"han"}
#   eosio.token <= eosio.token::transfer        {"from":"hodl","to":"han","quantity":"0.0001 SYS","memo":"Party! Your hodl is free."}
#          hodl <= eosio.token::transfer        {"from":"hodl","to":"han","quantity":"0.0001 SYS","memo":"Party! Your hodl is free."}
#           han <= eosio.token::transfer        {"from":"hodl","to":"han","quantity":"0.0001 SYS","memo":"Party! Your hodl is free."}
```
{% endcode %}
