---
description: 블록체인상의 데이터를 다중 인덱스(Multiple Indices) 로 저장하는 방법
---

# 보조 인덱스(Secondary Indices)

이 단원에서는 스마트 컨트랙트가 사용하는 데이터를 어떻게 지속적으로 유지하는지 설명하고, 이러한 데이터에 접근하기 위한 복수의 인덱스를 만들어 보겠습니다.

## 개요

Antelope 은 최대 16개까지의 인덱스를 사용하여 테이블을 정렬 할 수 있습니다. 이 단원에서는 `addressbook` 컨트랙트에 또다른 인덱스를 추가한 다음 이 인덱스를 이용하여 레코드를 순회하는 방법을 알아보겠습니다.

## 단계1: 테이블에서 기존 데이터 제거

앞서 언급했듯이, 테이블이 데이터를 가지고 있다면 테이블 구조를 수정할 수 없습니다. 따라서 이전 단원에서 추가했던 계정 alice 와 bob 의 모든 레코드를 삭제하겠습니다.

```cpp
$ cleos push action addressbook erase '["alice"]' -p alice@active
$ cleos push action addressbook erase '["bob"]' -p bob@active
```

## 단계2: 새 인덱스 멤버 및 getter 추가

`addressbook.cpp` 컨트랙트의 `person` 구조체에 새로운 멤버 변수 및 `getter` 를 추가합니다. 보조 인덱스가 되는 `age` 변수는 숫자형 필드이기 때문에 `uint64_t` 타입으로 정의하였습니다.

```cpp
uint64_t age;
uint64_t get_secondary_1() const { return age;} 
```

## 단계3: address 테이블 구성에 보조 인덱스 추가

보조 인덱스로 사용할 필드를 추가하였고, 다음으로 `address_index` 테이블을 수정해야 합니다.

{% code overflow="wrap" %}
```cpp
using address_index = eosio::multi_index<"people"_n, person,
indexed_by<"byage"_n, const_mem_fun<person, uint64_t, &person::get_secondary_1>>
>;
```
{% endcode %}

인덱스를 인스턴스화하는 데 사용할 `indexed_by` 구조체를 세 번째 매개 변수로서 전달합니다.

`indexed_by` 구조체에서 첫 번째 파라미터인 인덱스 이름을 "byage"로 지정하고, 두 번째 타입 파라미터는 인덱스 키로 상수 값을 추출하는 함수 호출 연산자로 지정합니다. 여기서는 앞에서 생성한 `getter` 를 가리키므로 age 변수를 사용하여 이 다중 인덱스 테이블의 레코드를 인덱싱 할 수 있게 됩니다.

```cpp
indexed_by<"byage"_n, const_mem_fun<person, uint64_t, &person::get_secondary_1>>
```

## 단계4: 코드 수정

이전 단계에서 수정한 내용을 가지고 이제 upsert 함수를 업데이트 하겠습니다. 아래와 같이 upsert 함수의 매개변수에 age 를 추가합니다.

{% code overflow="wrap" %}
```cpp
void upsert(name user, std::string first_name, std::string last_name, uint64_t age, std::string street, std::string city, std::string state)
```
{% endcode %}

upsert 함수의 age 필드를 업데이트 하기 위해 다음과 같이 추가 코드를 작성합니다.

{% code overflow="wrap" %}
```cpp
void upsert(name user, std::string first_name, std::string last_name, uint64_t age, std::string street, std::string city, std::string state) {
  require_auth( user );
  address_index addresses( get_first_receiver(), get_first_receiver().value);
  auto iterator = addresses.find(user.value);
  if( iterator == addresses.end() )
  {
    addresses.emplace(user, [&]( auto& row ) {
      row.key = user;
      row.first_name = first_name;
      row.last_name = last_name;
      // -- Add code below --
      row.age = age;
      row.street = street;
      row.city = city;
      row.state = state;
    });
  }
  else {
    addresses.modify(iterator, user, [&]( auto& row ) {
      row.key = user;
      row.first_name = first_name;
      row.last_name = last_name;
      // -- Add code below --
      row.age = age;
      row.street = street;
      row.city = city;
      row.state = state;
    });
  }
}
```
{% endcode %}

## 단계5: 컴파일 및 배포

코드가 완성 되었으니 다음과 같이 컴파일 하고 배포 합니다.

```cpp
cdt-cpp --abigen addressbook.cpp -o addressbook.wasm
```

```cpp
cleos set contract addressbook CONTRACTS_DIR/addressbook
```

## 단계6: 테스트

레코드를 삽입해 봅시다.

{% code overflow="wrap" %}
```cpp
$ cleos push action addressbook upsert '["alice", "alice", "liddell", 9, "123 drink me way", "wonderland", "amsterdam"]' -p alice@active

$ cleos push action addressbook upsert '["bob", "bob", "is a guy", 49, "doesnt exist", "somewhere", "someplace"]' -p bob@active
```
{% endcode %}

인덱스 age 를 가지고 alice 의 주소를 찾아보겠습니다. 다음 명령에서 볼 수 있는 "--index 2" 매개변수는 보조 인덱스를 사용한다는 것을 나타냅니다.

{% code overflow="wrap" %}
```cpp
$ cleos get table addressbook addressbook people --upper 10 \\
--key-type i64 \\
--index 2

{
  "rows": [{
      "key": "alice",
      "first_name": "alice",
      "last_name": "liddell",
      "age": 9,
      "street": "123 drink me way",
      "city": "wonderland",
      "state": "amsterdam"
    }
  ],
  "more": false,
  "next_key": ""
}
```
{% endcode %}

다음과 같이 bob 의 나이 범위까지 찾아보겠습니다.

{% code overflow="wrap" %}
```cpp
$ cleos get table addressbook addressbook people --upper 50 --key-type i64 --index 2

{
  "rows": [{
      "key": "alice",
      "first_name": "alice",
      "last_name": "liddell",
      "age": 9,
      "street": "123 drink me way",
      "city": "wonderland",
      "state": "amsterdam"
    },{
      "key": "bob",
      "first_name": "bob",
      "last_name": "is a guy",
      "age": 49,
      "street": "doesnt exist",
      "city": "somewhere",
      "state": "someplace"
    }
  ],
  "more": false
}
```
{% endcode %}

### 요약정리

지금까지의 작성한 전체 `addressbook` 컨트랙트 코드 내용은 다음과 같습니다.

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
    require_auth( user );
    address_index addresses(get_first_receiver(),get_first_receiver().value);
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
    }
    else {
      addresses.modify(iterator, user, [&]( auto& row ) {
        row.key = user;
        row.first_name = first_name;
        row.last_name = last_name;
        row.age = age;
        row.street = street;
        row.city = city;
        row.state = state;
      });
    }
  }

  [[eosio::action]]
  void erase(name user) {
    require_auth(user);

    address_index addresses(get_self(), get_first_receiver().value);

    auto iterator = addresses.find(user.value);
    check(iterator != addresses.end(), "Record does not exist");
    addresses.erase(iterator);
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
    uint64_t get_secondary_1() const { return age; }

  };

  using address_index = eosio::multi_index<"people"_n, person, indexed_by<"byage"_n, const_mem_fun<person, uint64_t, &person::get_secondary_1>>>;

};
```
{% endcode %}
