# 인라인 액션을 추가하는 법

## 개요

이 단원에서는 스마트 컨트랙트에서 인라인 액션을 호출하는 방법을 학습할 것입니다.

이전 단원에서 `addressbook` 컨트랙트를 작성하고 다중 인덱스 테이블을 사용하는 기본적인 방법을 알아보았습니다. 본편에서는 액션을 구성하는 방법과 컨트랙트 안에서 이러한 액션을 보내는 방법을 배울 것입니다.

## 단계1: 권한에 eosio.code 를 추가

`addressbook` 에서 인라인 액션을 보내려면 이 컨트랙트를 가지고 있는 계정의 active 권한에 `eosio.code` 권한을 추가해야 합니다. 터미널을 열고 다음 코드를 실행합니다.

```cpp
cleos set account permission addressbook active --add-code
```

`eosio.code` 권한은 보안을 강화하고 인라인 액션을 실행하는 컨트랙트를 활성화하기 위해 구현된 의사(Pseudo) 권한입다.

## 단계2: 액션 알림

이전 단원에서 작성한 `addressbook.cpp` 파일을 엽니다. 그리고 `addressbook` 컨트랙트에서 트랜잭션이 발생할 때마다 "트랜잭션 영수증"을 발송하는 helper 함수인 `notify` 액션을 작성하겠습니다.

```
[[eosio::action]]
void notify(name user, std::string msg) {}
```

이 함수는 user 계정을 `name` 타입으로, 메시지를 `string` 타입으로 받는 단순한 함수입니다. user 는 메시지를 받을 사용자를 나타냅니다.

## 단계3: require\_recipient 를 사용하여 발신자에게 액션을 복사

이 트랜잭션은 영수증으로 활용 될 수 있도록 `require_recipient` 메서드를 사용하여 발신한 사용자에게도 복사되어야 합니다. `require_recipient` 를 호출하면 `require_recipient` set 에 계정이 추가되고, 추가된 계정이 실행 중인 액션의 알림을 수신하도록 합니다. 이 알림은 `require_recipient` set 에 들어 있는 계정으로 액션의 "carbon copy"를 보내는 것과 같습니다.

```
[[eosio::action]]
  void notify(name user, std::string msg) {
   require_recipient(user);
  }
```

이 액션은 매우 간단하지만 모든 사용자가 이 기능을 호출할 수 있으며, 이 컨트랙트에서 받은 영수증을 위조 할 수도 있는 문제점이 있습니다. 이는 악의적인 수단으로 사용될 수 있기 때문에 취약성으로 간주되어야 합니다. 이 문제를 해결하려면 `get_self` 를 사용하여, 이 액션을 호출할 수 있는 권한을 컨트랙트 자체적으로 제공하도록 합니다.

```
[[eosio::action]]
  void notify(name user, std::string msg) {
    require_auth(get_self());
    require_recipient(user);
  }
```

이제 만약 사용자 bob 이 함수를 직접적으로 호출할 때 매개변수로 alice 를 전달하면 예외가 발생하게 됩니다.

## 단계4: 인라인 트랜잭션 전송을 위하여 helper 에게 알림

이 인라인 액션은 여러 번 호출될 것이므로 코드를 최대한 재사용 할 수 있도록 helper 메소드를 작성하겠습니다. 컨트랙트의 `private` 영역에 다음과 같이 새 메소드를 정의합니다.

```
...
  private:
    void send_summary(name user, std::string message){}
```

이제 이 helper 의 내부에 액션을 만들고 이 액션을 보낼 것입니다.

## 단계5: 액션 생성자

사용자가 컨트랙트의 액션을 수행 할 때마다 영수증을 사용자에게 보내도록 주소록 컨트랙트를 수정합니다.

테이블에 레코드를 만들 때를 다시 떠올려 봅시다. 테이블에 레코드가 없을 때 라는 것은, 즉 `iterator == address.ends()`가 true 인 경우입니다.

이 오브젝트를 `notification` 이라는 액션 변수에 저장합니다.

```
...
  private:
    void send_summary(name user, std::string message){
      action(
        //permission_level,
        //code,
        //action,
        //data
      );
    }
```

액션 생성자에 다음과 같은 매개 변수가 필요합니다.

* ``[`permission_level`](https://developers.eos.io/manuals/eosio.cdt/latest/structeosio\_1\_1permission\_\_level) 구조체
* 호출할 컨트랙트 코드 (`eosio::name` 타입을 사용하여 초기화)
* 액션 (`eosio:name` 타입을 사용하여 초기화됨)
* 액션에 전달할 데이터, 호출되는 액션과 관련된 위치들의 튜플.

### 권한 구조체

이 컨트렉트는 `get_self()`를 사용하여 컨트랙트 자체의 active 권한에 의해 승인되어야 합니다. 위에서 설명한대로 계약에서 `eosio.code` 의사 권한(pseudo-authority)에 active 권한을 부여하려면 active 권한을 인라인으로 사용해야 합니다.

```
...
  private:
    void send_summary(name user, std::string message){
      action(
        permission_level{get_self(),"active"_n},
        //code,
        //action,
        //data
      );
    }
```

### Code - 계약이 배포되어 있는 계정

호출된 액션이 이 컨트랙트 안에 있으므로 `get_self()`를 사용합니다. `"addressbook"_n`도 가능은 하지만 이 컨트랙트가 다른 계정 이름으로 배포되면 작동하지 않기 때문에 `get_self()` 를 사용하는 것이 더 낫습니다.

```
...
  private:
    void send_summary(name user, std::string message){
      action(
        permission_level{get_self(),"active"_n},
        get_self(),
        //action
        //data
      );
    }
```

## 액션

`notify` 액션은 이전에 이 인라인 액션에서 호출되도록 정의되었습니다. 여기서는 `_n` 연산자를 사용합니다.

```
...
  private:
    void send_summary(name user, std::string message){
      action(
        permission_level{get_self(),"active"_n},
        get_self(),
        "notify"_n,
        //data
      );
    }
```

## 데이터

마지막으로 이 액션에 전달할 데이터를 정의합니다. 알림 함수는 `name` 과 문자열의 두 매개 변수를 받습니다. 액션 생성자는 데이터를 bytes 타입으로 받기 때문에 std C++ 라이브러리를 통해 사용할 수 있는 함수인 `make_tuple` 을 사용할 것입니다. 이 튜플로부터 전달된 데이터는 positional 하며 호출되는 액션에서 받는 매개 변수의 순서에 따라 결정됩니다.

* upsert() 액션의 매개 변수로 제공되는 user 변수를 전달합니다.
* user 의 name 이 포함된 문자열에 notify 액션에 전달할 message 문자열을 더합니다.

```
...
  private:
    void send_summary(name user, std::string message){
      action(
        permission_level{get_self(),"active"_n},
        get_self(),
        "notify"_n,
        std::make_tuple(user, name{user}.to_string() + message)
      );
    }
```

## 액션 보내기

마지막으로 액션 구조체의 send 메소드를 사용하여 액션을 전송합니다.

```
...
  private:
    void send_summary(name user, std::string message) {
      action(
        permission_level{get_self(),"active"_n},
        get_self(),
        "notify"_n,
        std::make_tuple(user, name{user}.to_string() + message)
      ).send();
    }
```

## 단계6: helper 호출 및 메시지 삽입

이제 helper 메소드가 정의되었습니다. 이 컨트랙트 코드의 다음과 같은 위치에서 새롭게 만든 helper 메소드를 호출할 것입니다.

* `emplaces` 가 새 레코드를 만든 다음:\
  `send_summary(user, "successfully emplaced record to addressbook");`
* `modify` 가 기존 레코드를 수정한 다음:\
  `send_summary(user, "successfully modified record in addressbook.");`
* `erase` 가 기존 레코드를 삭제한 다음:\
  `send_summary(user, "successfully erased record from addressbook");`

## 단계 7: 재 컴파일 및 ABI 파일 재생성

이제 준비가 다 되었습니다. 완성된 `addressbook` 컨트랙트는 다음과 같습니다.

{% code overflow="wrap" %}
```cpp
#include <eosio/eosio.hpp>
#include <eosio/print.hpp>

using namespace eosio;

class [[eosio::contract("addressbook")]] addressbook : public eosio::contract {

public:

  addressbook(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

  [[eosio::action]]
  void upsert(name user, std::string first_name, std::string last_name, uint64_t age, std::string street, std::string city, std::string state) {
    require_auth(user);
    address_index addresses(get_first_receiver(), get_first_receiver().value);
    auto iterator = addresses.find(user.value);
    if( iterator == addresses.end() )
    {
      addresses.emplace(user, [&]( auto& row ) {
       row.key = user;
       row.first_name = first_name;
       row.last_name = last_name;
       row.age = age;
       row.street = street;
       row.city = city;
       row.state = state;
      });
      send_summary(user, " successfully emplaced record to addressbook");
    }
    else {
      addresses.modify(iterator, user, [&]( auto& row ) {
        row.key = user;
        row.first_name = first_name;
        row.last_name = last_name;
        row.street = street;
        row.city = city;
        row.state = state;
      });
      send_summary(user, " successfully modified record to addressbook");
    }
  }

  [[eosio::action]]
  void erase(name user) {
    require_auth(user);

    address_index addresses(get_first_receiver(), get_first_receiver().value);

    auto iterator = addresses.find(user.value);
    check(iterator != addresses.end(), "Record does not exist");
    addresses.erase(iterator);
    send_summary(user, " successfully erased record from addressbook");
  }

  [[eosio::action]]
  void notify(name user, std::string msg) {
    require_auth(get_self());
    require_recipient(user);
  }

private:
  struct [[eosio::table]] person {
    name key;
    std::string first_name;
    std::string last_name;
    uint64_t age;
    std::string street;
    std::string city;
    std::string state;

    uint64_t primary_key() const { return key.value; }
    uint64_t get_secondary_1() const { return age;}
  };

  void send_summary(name user, std::string message) {
    action(
      permission_level{get_self(),"active"_n},
      get_self(),
      "notify"_n,
      std::make_tuple(user, name{user}.to_string() + message)
    ).send();
  };

  typedef eosio::multi_index<"people"_n, person,
    indexed_by<"byage"_n, const_mem_fun<person, uint64_t, &person::get_secondary_1>>
  > address_index;
};
```
{% endcode %}

터미널에서 `CONTRACTS_DIR/addressbook` 으로 이동합니다.

```cpp
cd CONTRACTS_DIR/addressbook
```

본문에서 변경한 코드가 ABI 에 영향을 주기 때문에 `--abigen` 플래그를 넣어서 컨트랙트를 다시 컴파일 합니다.

```cpp
$ cdt-cpp -o addressbook.wasm addressbook.cpp --abigen
```

변경된 컨트랙트를 다시 배포하면 온체인에 올라간 스마트 컨트랙트가 업그레이드 됩니다.

{% code overflow="wrap" %}
```cpp
$ cleos set contract addressbook CONTRACTS_DIR/addressbook

Publishing contract...
executed transaction: 1898d22d994c97824228b24a1741ca3bd5c7bc2eba9fea8e83446d78bfb264fd  7320 bytes  747 us
#         eosio <= eosio::setcode               {"account":"addressbook","vmtype":0,"vmversion":0,"code":"0061736d0100000001a6011a60027f7e0060077f7e...
#         eosio <= eosio::setabi                {"account":"addressbook","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d65.
```
{% endcode %}

## 단계8: 테스트

이제 컨트랙트가 수정되고 배포되었으니 테스트 해보겠습니다. 이전 단원의 테스트 단계에서 alice 의 addressbook 레코드를 삭제했었습니다. 따라서 터미널에서 다음과 같이 "create" 케이스 내부에 작성된 upsert 인라인 액션을 호출합니다.

{% code overflow="wrap" %}
```shell
$ cleos push action addressbook upsert '["alice", "alice", "liddell", 21, "123 drink me way", "wonderland", "amsterdam"]' -p alice@active

executed transaction: e9e30524186bb6501cf490ceb744fe50654eb393ce0dd733f3bb6c68ff4b5622  160 bytes  9810 us
#   addressbook <= addressbook::upsert          {"user":"alice","first_name":"alice","last_name":"liddell","age":21,"street":"123 drink me way","cit...
#   addressbook <= addressbook::notify          {"user":"alice","msg":"alicesuccessfully emplaced record to addressbook"}
#         alice <= addressbook::notify          {"user":"alice","msg":"alicesuccessfully emplaced record to addressbook"}
```
{% endcode %}

위에 출력된 로그의 마지막 항목은 alice 로 보내진 `addressbook::notify` 액션을 나타냅니다. `cleos get actions` 명령으로 alice와 관련된 액션들을 표시해 봅시다.

{% code overflow="wrap" %}
```shell
$ cleos get actions alice

#  seq  when                              contract::action => receiver      trx id...   args
================================================================================================================
#   62   2018-09-15T12:57:09.000       addressbook::notify => alice         685ecc09... {"user":"alice","msg":"alice successfully added record to ad...
```
{% endcode %}
