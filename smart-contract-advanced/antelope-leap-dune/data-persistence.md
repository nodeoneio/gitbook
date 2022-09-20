# 데이터 지속성(Data Persistence)

## 개요

본 문서에서는 블록체인에서 스마트 컨트랙트에 사용하는 데이터를 유지하는 방법을 배울 것입니다.

여기서는 데이터 지속성의 예를 보여주고 간단한 주소록 스마트 컨트랙트를 만들 것입니다. 구현하게 될 내용이 프로덕션용으로 쓸만한 스마트 컨트랙트는 아니지만, `eosio` 의 `multi_index` API 기능과 관련이 없는 비즈니스 로직을 배제하고 Antelope 에서 데이터 지속성이 작동하는 방식을 배울 수 있는 괜찮은 스마트 컨트랙트 예제입니이다.

## 단계1: 새 디렉토리 생성

명령줄에서 컨트랙트 작업용 디렉토리를 만들고 들어갑니다.

```jsx
mkdir CONTRACTS_DIR 
cd CONTRACTS_DIR
```

그리고 주소록 디렉토리를 만들고 들어갑니다.

```jsx
mkdir addressbook
cd addressbook
```

## 단계2: 새 파일 작성

```jsx
touch addressbook.cpp
```

위 명령을 실행하고 만들어진 빈 파일을 엽니다.

## 단계3: 컨트랙트 클래스 작성

`addressbook.cpp` 파일에 `addressbook` 클래스를 작성합니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract("addressbook")]] addressbook : public eosio::contract { 
  public:

  private:

};
```

## 단계4: 데이터 구조 작성

테이블이 설정되고 인스턴스화 되기 전에 `addressbook` 의 데이터 구조를 표현할 수 있는 구조체를 정의해야 합니다. 주소록 프로그램이기 때문에 테이블은 사람의 정보를 가지는 `people` 구조체를 만들겠습니다. 이 구조체를 `private` 영역에 하나 만듭니다.

```cpp
private:
    struct person {};
```

멀티인덱스 테이블의 구조를 정의 할 때는 중복되지 않는 유일한 값을 Primary Key로 사용해야 합니다.

이 컨트랙트에서는 사용자의 이름에 기반하여 `name` 타입의 `key` 라는 필드를 사용할 것입니다. 이 예제에서 하나의 사용자는 하나의 고유한 항목이 될 것이므로 이 `key` 값이 유일한 값이 됩니다.

```cpp
struct person {
 name key;
};
```

이 컨트랙트는 주소록이므로, 주소와 관련된 정보들을 `person` 구조체에 저장하겠습니다.

```cpp
struct person {
 name key;
 std::string first_name;
 std::string last_name;
 std::string street;
 std::string city;
 std::string state;
};
```

이제 기본 데이터 구조가 완성되었습니다.

다음으로 `primary_key()` 메소드를 정의합니다. 모든 `multi_index` 구조체는 `primary_key()` 메소드가 필요합니다. 이 메소드는 `multi_index` 인스턴스의 인덱스 상세 내용에 따라 백그라운드에서 사용됩니다. Antelope 의 `multi_index` 는 boost 라이브러리의 [multi\_index](https://www.boost.org/doc/libs/1\_59\_0/libs/multi\_index/doc/index.html) 를 래핑한 데이터베이스 입니다.

아래와 같이 `primary_key()` 메서드를 생성하고 구조체 멤버(이 경우 앞에서 설명한 대로 key 필드)를 반환합니다.

```cpp
struct person {
 name key;
 std::string first_name;
 std::string last_name;
 std::string street;
 std::string city;
 std::string state;

 uint64_t primary_key() const { return key.value;}
};
```

{% hint style="info" %}
테이블이 데이터를 가지고 있을 경우 그 테이블의 데이터 구조를 바꿀 수 없습니다. 만약 테이블 구조를 바꿔야 할 경우가 생기면 먼저 모든 데이터를 지워야 합니다.
{% endhint %}

## 단계5: 멀티인덱스 테이블 구성

이제 구조체를 사용하여 테이블의 데이터 구조를 정의하였으므로 테이블을 구성할 차례입니다. `eosio::multi_index` 생성자에 테이블 이름과 앞서 정의한 구조체를 매개변수로 넘깁니다.

```cpp
using address_index = eosio::multi_index<"people"_n, person>;

or

typedef eosio::multi_index<"people"_n, person> address_index;
```

위와 같이 `multi_index` 구성을 사용하여 다음과 같은 속성을 가지는 `people` 이라는 테이블을 만들었습니다.

1. 테이블 이름에 `_n` 연산자를 붙여 `eosio:name` 타입임을 정의하고 이를 테이블의 이름으로 지정했습니다. 이 테이블에는 여러 개의 서로 다른 "person"이 포함되어 있으므로, 테이블의 이름을 `person` 의 복수형인 "people"으로 하였습니다.
2. 이전 단계에서 정의한 고유한 사용자 구조체인 `person` 을 전달합니다.
3. 이 테이블의 타입을 선언합니다. 이 타입은 나중에 이 테이블을 인스턴스화하는 데 사용됩니다.

지금까지 만든 소스 파일 내용은 다음과 같습니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract("addressbook")]] addressbook : public eosio::contract {

  public:

  private:
    struct [[eosio::table]] person {
      name key;
      std::string first_name;
      std::string last_name;
      std::string street;
      std::string city;
      std::string state;

      uint64_t primary_key() const { return key.value;}
    };
    
  using address_index = eosio::multi_index<"people"_n, person>;
};
```

## 단계6: 생성자

C++ 클래스를 작성할 때 가장 먼저 만들어야 하는 퍼블릭 메서드는 생성자(Constructor)입니다.

스마트 컨트랙트를 초기화 하는 책임은 생성자에게 있습니다.

Antelope 컨트랙트는 contract 클래스를 확장합니다. 컨트랙트와 수신자의 코드명을 사용하여 상위 컨트랙트 클래스를 초기화 합니다. 여기서 중요한 파라미터는 컨트랙트가 배포된 블록체인 계정을 나타내는 "code" 매개변수 입니다.

```cpp
addressbook(name receiver, name code, datastream<const char*> ds):contract(receiver, code, ds) {}
```

## 단계7: 테이블에 레코드 추가

앞서 만든 다중 인덱스 테이블의 Primary Key 는 이 컨트랙트에 사용자당 하나의 레코드만 저장하도록 정의되었습니다. 설계한 것들이 제대로 작동하기 위해서는 다음과 같이 몇 가지 규칙을 만들어야 합니다.

1. 주소록을 수정할 수 있는 유일한 계정은 사용자 자신이다.
2. 테이블의 primary\_key는 고유한 사용자 이름을 값으로 가진다.
3. 컨트랙트를 쉽게 사용하기 위해 하나의 액션 안에서 테이블에 데이터를 만들고 수정할 수 있는 기능이 있어야 한다.

Antelope 블록 체인에서 계정 이름은 유일한 값입니다. 그래서 `name` 타입은 primary\_key로서 적합한 대상입니다. 실제 내부적으로 `name` 타입은 `uint64_t` 정수값을 갖습니다.

이제 사용자가 레코드를 추가하거나 업데이트 할 수 있는 액션을 정의합니다. 이 액션은 어떤 데이터를 생성하거나 수정하려고 할 때 필요한 모든 값을 받을 수 있어야 합니다.

사용자 경험 및 인터페이스를 단순화 하기 위해 하나의 메소드로 테이블의 데이터 삽입 및 수정 작업을 모두 수행할 수 있도록 만들겠습니다. 그래서 이러한 동작을 한다는 의미로 "update"와 "insert"의 조합인 "upsert"를 액션 이름으로 하였습니다.

```cpp
void upsert(
  name user,
  std::string first_name,
  std::string last_name,
  std::string street,
  std::string city,
  std::string state
) {}
```

앞서 이 컨트랙트는 본인만이 자신의 데이터를 건드릴 수 있어야 한다고 이야기 한 바 있습니다. 따라서 `eosio.cdt` 에서 제공하는 `require_auth` 메소드를 사용할 것입니다. 이 메서드는 `name` 타입 인수를 받으며, 이 값이 트랜잭션을 실행하는 계정으로부터 넘겨 받은 값과 동일하면 적절한 사용 권한을 가지고 있다고 판단합니다.

```cpp
void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
  require_auth( user );
}
```

앞 단계에서 `multi_index` 테이블을 구성하였으며, 이름을 `address_index` 라고 선언하였습니다. 이 테이블을 인스턴스화하려면 두 개의 매개 변수가 필요합니다.

1. 첫 번째 매개 변수는 "code" 이며 이 테이블의 소유자 계정을 의미합니다. 소유자로서, 그 계정은 데이터 보관 비용을 지불해야 합니다. 별도로 다른 지급인(payer)을 지정하지 않는 한 code 만이 이 테이블의 데이터를 수정하거나 삭제할 수 있습니다. 여기서는 이 컨트랙트로 이름을 전달하기 위하여 `get_self()` 함수를 사용했습니다.
2. 두 번째 매개 변수 "scope"는 이 컨트랙트의 scope 범위 안에서 테이블의 고유성을 보장합니다. 본 예제의 경우 테이블이 하나만 있으므로 `get_first_receiver()` 값을 사용할 수 있습니다. `get_first_receiver` 함수에서 반환되는 값은 이 컨트랙트를 배포할 계정 이름입니다.

scope 는 다중 인덱스 테이블 내에서 테이블을 논리적으로 분리하는 데 사용합니다. (테이블 상에서 토큰 소유자의 범위를 제한하는 `eosio.token` 컨트랙트 `multi-index` 참조). scope 는 원래 각각의 하위 테이블에서 병렬 계산을 수행하기 위해 테이블의 상태를 분리하려는 의도로 만든 것이었습니다. 그러나 현재 블록체인 간 통신이 테이블 병렬화보다 우선시 되고 있기 때문에 scope 는 `eosio.token` 의 경우처럼 테이블을 논리적으로 분리하는 데만 사용되고 있습니다.

```cpp
void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
  require_auth( user );
  address_index addresses(get_self(), get_first_receiver().value);
}
```

다음으로 반복자(iterator)를 가져옵니다. 이 반복자는 나중에 다시 사용할 것이기 때문에 변수에 저장합니다.

```cpp
void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
  require_auth( user );
  address_index addresses(get_self(), get_first_receiver().value);
  auto iterator = addresses.find(user.value);
}
```

이제 실행 권한이 설정되고 테이블이 인스턴스화 되었습니다. 이제 테이블을 만들거나 수정하는 코드를 작성하겠습니다.

먼저 테이블의 `find` 메소드에 특정 사용자를 매개 변수로 넘겨, 테이블에 해당 사용자가 이미 있는지 여부를 확인합니다. `find` 메서드는 반복자를 반환합니다. 반환자는 `end()` 메소드의 반환값("null" 값과 일치합니다)와 비교하여 사용자의 유무를 확인할 수 있습니다.

```cpp
void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
  require_auth( user );
  address_index addresses(get_self(), get_first_receiver().value);
  auto iterator = addresses.find(user.value);
  if( iterator == addresses.end() )
  {
    //The user isn't in the table
  }
  else {
    //The user is in the table
  }
}
```

`multi_index` 의 메소드 `emplace` 를 사용하여 테이블에 레코드를 만듭니다. 이 메소드는 이 레코드 데이터가 차지하는 스토리지 사용량을 지불할 "payer" 계정과 콜백 함수 두 개의 인자를 받습니다.

`emplace` 메서드에 전달할 콜백 함수는 참조자(reference)를 만들기 위하여 람다 함수를 사용해야 합니다. 아래 코드에서 upsert 메소드에 인자로 전달된 값을 행의 값으로 할당하고 있습니다.

```cpp
void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
  require_auth( user );
  address_index addresses(get_self(), get_first_receiver().value);
  auto iterator = addresses.find(user.value);
  if( iterator == addresses.end() )
  {
    addresses.emplace(user, [&]( auto& row ) {
      row.key = user;
      row.first_name = first_name;
      row.last_name = last_name;
      row.street = street;
      row.city = city;
      row.state = state;
    });
  }
  else {
    //The user is in the table
  }
}
```

다음으로 `multi_index` 의 `modify` 메소드를 사용하여 upsert 에 수정/업데이트 기능을 추가하겠습니다. 다음과 같은 인자가 필요합니다.

* 앞서 받아온 반복자. 이 액션이 호출할 때 선언된 대로, 현재 사용자가 설정되어 있다.
* 이 데이터의 저장 비용을 지불하게 될 결제자 "payer". 이 코드에서는 사용자 자신이 된다.
* 실제로 행을 수정하는 콜백 함수.

```cpp
void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
  require_auth( user );
  address_index addresses(get_self(), get_first_receiver().value);
  auto iterator = addresses.find(user.value);
  if( iterator == addresses.end() )
  {
    addresses.emplace(user, [&]( auto& row ) {
      row.key = user;
      row.first_name = first_name;
      row.last_name = last_name;
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
      row.street = street;
      row.city = city;
      row.state = state;
    });
  }
}
```

이제 `addressbook` 컨트랙트는, 어떤 레코드가 아직 존재하지 않는다면 테이블에 추가하고, 해당 레코드가 이미 있는 경우에는 데이터를 수정하는 액션을 가지게 되었습니다.

이제 사용자의 레코드를 완전히 제거하는 방법을 알아보겠습니다.

## 단계8: 테이블에서 레코드 삭제

이전 단계와 마찬가지로, ABI 를 선언하고, 레코드의 소유자만 데이터를 수정할 수 있도록 `addressbook` 에 `erase()`라는 퍼블릭 메소드를 만들어 액션의 인수인 `user` 를 검증하는 `require_auth()` 를 수행합니다.

```cpp
void erase(name user){
      require_auth(user);
    }
```

이제 테이블을 인스턴스화 합니다. `addressbook` 에서 하나의 계정은 하나의 레코드만을 가지므로 `user` 값을 인수로 하는 `find` 메소드로 반복자를 가져옵니다.

```cpp
...
    void erase(name user){
      require_auth(user);
      address_index addresses(get_self(), get_first_receiver().value);
      auto iterator = addresses.find(user.value);
    }
...
```

존재하지 않는 레코드는 지울 수 없습니다. 따라서 먼저 레코드의 존재 여부를 확인해야 합니다.

```cpp
...
    void erase(name user){
      require_auth(user);
      address_index addresses(get_self(), get_first_receiver().value);
      auto iterator = addresses.find(user.value);
      check(iterator != addresses.end(), "Record does not exist");
    }
...
```

마지막으로 데이터를 지우기 위해 `multi_index` 의 `erase` 메소드를 호출합니다. 데이터가 지워지고 나면, 원래 payer 의 저장공간은 비워집니다.

```cpp
...
  void erase(name user) {
    require_auth(user);
    address_index addresses(get_self(), get_first_receiver().value);
    auto iterator = addresses.find(user.value);
    check(iterator != addresses.end(), "Record does not exist");
    addresses.erase(iterator);
  }
...
```

컨트랙트가 이제 거의 완성되어 갑니다. 이제 사용자는 레코드를 만들고 수정하고 삭제 까지 할 수 있게 되었습니다. 마지막으로 이 컨트랙트를 컴파일 해 봅시다.

## 단계9: ABI 준비

### 9.1 ABI 액션 선언

cdt 는 ABI 생성기를 가지고 있는데, 이를 사용하려면 몇 가지 속성 지정자 시퀀스(Attribute specifier sequence)가 필요합니다.

위에서 작성한 upsert 와 erase 두 함수 바로 윗줄에 다음과 같은 C++ 속성 지정자 시퀀스를 추가합니다.

```cpp
[[eosio::action]]
```

위의 속성 지정자 시퀀스는 생성된 ABI파일에서 액션의 인자들을 추출한 뒤 ABI 구조체 상세 내역들을 만듭니다.

### 9.2 ABI 테이블 선언

ABI 속성 지정자 시퀀스를 테이블 구조체에 추가합니다. 컨트랙트 코드의 private 구역에 정의되어 있는 아래 문장을 수정합니다.

```cpp
struct person {
```

위 내용을 아래와 같이 바꿉니다.

```cpp
struct [[eosio::table]] person {
```

`[[eosio::table]]` 속성 지정자 시퀀스를 추가하면 테이블과 관련된 상세 내역이 ABI 파일에 추가됩니다.

이제 컨트랙트가 컴파일 될 준비가 되었습니다.

아래가 `addressbook` 컨트랙트의 최종 코드입니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract("addressbook")]] addressbook : public eosio::contract {

public:

  addressbook(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

  [[eosio::action]]
  void upsert(name user, std::string first_name, std::string last_name, std::string street, std::string city, std::string state) {
    require_auth( user );
    address_index addresses( get_self(), get_first_receiver().value );
    auto iterator = addresses.find(user.value);
    if( iterator == addresses.end() )
    {
      addresses.emplace(user, [&]( auto& row ) {
       row.key = user;
       row.first_name = first_name;
       row.last_name = last_name;
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
        row.street = street;
        row.city = city;
        row.state = state;
      });
    }
  }

  [[eosio::action]]
  void erase(name user) {
    require_auth(user);

    address_index addresses( get_self(), get_first_receiver().value);

    auto iterator = addresses.find(user.value);
    check(iterator != addresses.end(), "Record does not exist");
    addresses.erase(iterator);
  }

private:
  struct [[eosio::table]] person {
    name key;
    std::string first_name;
    std::string last_name;
    std::string street;
    std::string city;
    std::string state;
    uint64_t primary_key() const { return key.value; }
  };
  using address_index = eosio::multi_index<"people"_n, person>;
};
```

## 단계 10:(선택사항) 리카르디안 컨트랙트(Ricardian Contract) 준비

리카르디안 컨트랙트 없이 스마트 컨트랙트를 컴파일하면 각각의 액션에 대한 리카르디안 컨트랙트(Ricardian contract)가 없다고 하는 컴파일러 경고가 발생합니다.

```cpp
Warning, action <upsert> does not have a ricardian contract
Warning, action <erase> does not have a ricardian contract
```

위에서 만든 스마트 컨트랙트를 위한 리카르디안 컨트랙트를 작성하려면, [addressbook.contracts.md](http://addressbook.contracts.md) 라는 파일을 하나 만듭니다. 리카르디안 컨트랙트 이름의 처음 부분은 반드시 스마트 컨트랙트의 이름과 일치해야 합니다.

```cpp
touch addressbook.contract.md
```

리카르디안 컨트랙트의 정의를 이 파일에 작성합니다.

```cpp
<h1 class="contract">upsert</h1>
---
spec-version: 0.0.2
title: Upsert
summary: This action will either insert or update an entry in the address book. If an entry exists with the same name as the specified user parameter, the record is updated with the first_name, last_name, street, city, and state parameters. If a record does not exist, a new record is created. The data is stored in the multi index table. The ram costs are paid by the smart contract.
icon:

<h1 class="contract">erase</h1>
---
spec-version: 0.0.2
title: Erase
summary: This action will remove an entry from the address book if an entry in the multi index table exists with the specified name.
icon:
```

## 단계11:(선택사항) 리카르디안 구절(Ricardian clause) 준비

위에서 만든 스마트 컨트랙트를 위한 리카르디안 구절을 작성하려면, [addressbook.clauses.md](http://addressbook.contracts.md) 라는 파일을 하나 만듭니다. 마찬가지로 리카르디안 구절 이름의 첫 부분은 반드시 스마트 컨트랙트의 이름과 일치해야 합니다.

```cpp
touch addressbook.clauses.md
```

다음의 리카르디안 구절 정의를 위 파일에 붙여넣습니다.

```cpp
<h1 class="clause">Data Storage</h1>
---
spec-version: 0.0.1
title: General Data Storage
summary: This smart contract will store data added by the user. The user consents to the storage of this data by signing the transaction.
icon:

<h1 class="clause">Data Usage</h1>
---
spec-version: 0.0.1
title: General Data Use
summary: This smart contract will store user data. The smart contract will not use the stored data for any purpose outside store and delete.
icon:

<h1 class="clause">Data Ownership</h1>
---
spec-version: 0.0.1
title: Data Ownership
summary: The user of this smart contract verifies that the data is owned by the smart contract, and that the smart contract can use the data in accordance to the terms defined in the Ricardian Contract.
icon:

<h1 class="clause">Data Distirbution</h1>
---
spec-version: 0.0.1
title: Data Distirbution
summary: The smart contract promises to not actively share or distribute the address data. The user of the smart contract understands that data stored in a multi index table is not private data and can be accessed by any user of the blockchain.  
icon:

<h1 class="clause">Data Future</h1>
---
spec-version: 0.0.1
title: Data Future
summary: The smart contract promises to only use the data in accordance of the terms defined in the Ricardian Contract, now and at all future dates.
icon:
```

## 단계 12: 계약 컴파일

터미널에서 다음 명령을 입력합니다.

```cpp
cdt-cpp addressbook.cpp -o addressbook.wasm
```

리카르디안 컨트랙트와 구절을 만들었다면, 그 내용이 `.abi` 파일에 표시될 것입니다. 위에서 작성한 예제 `addressbook.cpp` 파일의 내용과 그에 해당하는 리카르디안 컨트랙트와 구절을 적용한 ABI 파일은 다음과 같습니다.

```cpp
{
    "____comment": "This file was generated with eosio-abigen. DO NOT EDIT ",
    "version": "eosio::abi/1.1",
    "types": [],
    "structs": [
        {
            "name": "erase",
            "base": "",
            "fields": [
                {
                    "name": "user",
                    "type": "name"
                }
            ]
        },
        {
            "name": "person",
            "base": "",
            "fields": [
                {
                    "name": "key",
                    "type": "name"
                },
                {
                    "name": "first_name",
                    "type": "string"
                },
                {
                    "name": "last_name",
                    "type": "string"
                },
                {
                    "name": "street",
                    "type": "string"
                },
                {
                    "name": "city",
                    "type": "string"
                },
                {
                    "name": "state",
                    "type": "string"
                }
            ]
        },
        {
            "name": "upsert",
            "base": "",
            "fields": [
                {
                    "name": "user",
                    "type": "name"
                },
                {
                    "name": "first_name",
                    "type": "string"
                },
                {
                    "name": "last_name",
                    "type": "string"
                },
                {
                    "name": "street",
                    "type": "string"
                },
                {
                    "name": "city",
                    "type": "string"
                },
                {
                    "name": "state",
                    "type": "string"
                }
            ]
        }
    ],
    "actions": [
        {
            "name": "erase",
            "type": "erase",
            "ricardian_contract": "---\\nspec-version: 0.0.2\\ntitle: Erase\\nsummary: his action will remove an entry from the address book if an entry exists with the same name \\nicon:"
        },
        {
            "name": "upsert",
            "type": "upsert",
            "ricardian_contract": "---\\nspec-version: 0.0.2\\ntitle: Upsert\\nsummary: This action will either insert or update an entry in the address book. If an entry exists with the same name as the user parameter the record is updated with the first_name, last_name, street, city and state parameters. If a record does not exist a new record is created. The data is stored in the multi index table. The ram costs are paid by the smart contract.\\nicon:"
        }
    ],
    "tables": [
        {
            "name": "people",
            "type": "person",
            "index_type": "i64",
            "key_names": [],
            "key_types": []
        }
    ],
    "ricardian_clauses": [
        {
            "id": "Data Storage",
            "body": "---\\nspec-version: 0.0.1\\ntitle: General data Storage\\nsummary: This smart contract will store data added by the user. The user verifies they are happy for this data to be stored.\\nicon:"
        },
        {
            "id": "Data Usage",
            "body": "---\\nspec-version: 0.0.1\\ntitle: General data Use\\nsummary: This smart contract will store user data. The smart contract will not use the stored data for any purpose outside store and delete \\nicon:"
        },
        {
            "id": "Data Ownership",
            "body": "---\\nspec-version: 0.0.1\\ntitle: Data Ownership\\nsummary: The user of this smart contract verifies that the data is owned by the smart contract, and that the smart contract can use the data in accordance to the terms defined in the Ricardian Contract \\nicon:"
        },
        {
            "id": "Data Distirbution",
            "body": "---\\nspec-version: 0.0.1\\ntitle: Data Ownership\\nsummary: The smart contract promises to not actively share or distribute the address data. The user of the smart contract understands that data stored in a multi index table is not private data and can be accessed by any user of the blockchain.  \\nicon:"
        },
        {
            "id": "Data Future",
            "body": "---\\nspec-version: 0.0.1\\ntitle: Data Ownership\\nsummary: The smart contract promises to only use the data in accordance to the terms defined in the Ricardian Contract, now and at all future dates. \\nicon:"
        }
    ],
    "variants": []
}
```

## 단계 13: 컨트랙트 배포

다음 명령어를 실행하여 컨트랙트 배포에 필요한 계정을 하나 만듭니다.

```cpp
cleos create account eosio addressbook EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV -p eosio@active
```

이제 `addressbook` 컨트랙트를 온체인으로 배포합니다.

```cpp
$ cleos set contract addressbook CONTRACTS_DIR/addressbook -p addressbook@active

5f78f9aea400783342b41a989b1b4821ffca006cd76ead38ebdf97428559daa0  5152 bytes  727 us
#         eosio <= eosio::setcode               {"account":"addressbook","vmtype":0,"vmversion":0,"code":"0061736d010000000191011760077f7e7f7f7f7f7f...
#         eosio <= eosio::setabi                {"account":"addressbook","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d65...
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

## 단계 14: 컨트랙트 테스트

테이블에 행을 추가해 봅시다.

```cpp
$ cleos push action addressbook upsert '["alice", "alice", "liddell", "123 drink me way", "wonderland", "amsterdam"]' -p alice@active

executed transaction: 003f787824c7823b2cc8210f34daed592c2cfa66cbbfd4b904308b0dfeb0c811  152 bytes  692 us
#   addressbook <= addressbook::upsert          {"user":"alice","first_name":"alice","last_name":"liddell","street":"123 drink me way","city":"wonde...
```

계정 alice 는 다른 계정 이름을 가지고 레코드를 추가 할 수 없을 것입니다. 다음과 같이 확인해 봅시다.

```cpp
$ cleos push action addressbook upsert '["bob", "bob", "is a loser", "doesnt exist", "somewhere", "someplace"]' -p alice@active
```

예상대로 `require_auth()` 때문에 alice 는 다른 사용자 이름으로 레코드를 추가/수정 할 수 없습니다.

```cpp
Error 3090004: Missing required authority
Ensure that you have the related authority inside your transaction!;
If you are currently using 'cleos push action' command, try to add the relevant authority using -p option.
Error Details:
missing authority of bob
```

alice 의 레코드를 조회해 봅시다.

```cpp
$ cleos get table addressbook addressbook people --lower alice --limit 1

{ 
  "rows": [{
      "key": "alice",
      "first_name": "alice",
      "last_name": "liddell",
      "street": "123 drink me way",
      "city": "wonderland",
      "state": "amsterdam"
    }
  ],
  "more": false,
  "next_key": ""
}
```

alice 가 레코드를 삭제할 수 있는지 확인해 봅시다.

```cpp
$ cleos push action addressbook erase '["alice"]' -p alice@active

executed transaction: 0a690e21f259bb4e37242cdb57d768a49a95e39a83749a02bced652ac4b3f4ed  104 bytes  1623 us
#   addressbook <= addressbook::erase           {"user":"alice"}
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

레코드가 삭제되었는지 확인해 봅니다.

```cpp
$ cleos get table addressbook addressbook people --lower alice --limit 1

{
  "rows": [],
  "more": false，
  "next_key": ""
}
```

## 요약

본 단원에서 테이블 설정법과, 인스턴트화 하는 방법, 테이블에 새 레코드를 추가하는 방법, 기존 레코드를 수정하는 방법, 그리고 반복자를 사용하는 방법을 배웠습니다. 또한 반복자의 빈(empty) 결과를 테스트에 활용하는 방법도 배웠습니다.
