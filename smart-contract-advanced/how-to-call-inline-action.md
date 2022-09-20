# 다른 스마트 컨트랙트의 인라인 액션을 호출하는 방법

## 소개

이전 단원에서 스마트 컨트랙트의 액션에서 인라인 액션을 실행하는 방법을 알아보았습니다. 이번에는 외부에 있는 컨트랙트로 액션을 보내는 방법에 대하여 알아볼 것입니다.&#x20;

지금까지 컨트랙트를 제법 많이 작성했으므로, 이 컨트랙트는 매우 단순하게 유지 할 것입니다. 여기서 우리는 컨트랙트에 의해 작성된 액션을 카운트 하는 컨트랙트를 작성할 것입니다. 이 컨트랙트는 실제 상황에서 쓰일 일이 거의 없겠지만, 외부의 스마트 컨트랙트에 대한 인라인 액션 호출 방법의 예시를을 보여줄 것입니다.

## 단계1: Addressbook 카운팅 컨트랙트

`CONTRACTS_DIR` 로 이동합니다. 그리고 `abcounter` 라는 디렉토리를 만들고 `abcounter.cpp` 라는 파일을 만듭니다.

```cpp
cd CONTRACTS_DIR
mkdir abcounter
cd abcounter
touch abcounter.cpp
```

편집기로 `abcounter.cpp` 파일을 열고 다음 코드를 붙여넣습니다. 이 컨트랙트는 지금까지 다루었던 내용을 기반으로 매우 기본적인 내용으로 구성되어 있습니다. 몇 가지 예외가 있기는 하지만, 그 내용에 대해서는 아래에 자세히 설명할 것입니다.

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract("abcounter")]] abcounter : public eosio::contract {
  public:

    abcounter(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

    [[eosio::action]]
    void count(name user, std::string type) {
      require_auth( name("addressbook"));
      count_index counts(get_first_receiver(), get_first_receiver().value);
      auto iterator = counts.find(user.value);

      if (iterator == counts.end()) {
        counts.emplace("addressbook"_n, [&]( auto& row ) {
          row.key = user;
          row.emplaced = (type == "emplace") ? 1 : 0;
          row.modified = (type == "modify") ? 1 : 0;
          row.erased = (type == "erase") ? 1 : 0;
        });
      }
      else {
        counts.modify(iterator, "addressbook"_n, [&]( auto& row ) {
          if(type == "emplace") { row.emplaced += 1; }
          if(type == "modify") { row.modified += 1; }
          if(type == "erase") { row.erased += 1; }
        });
      }
    }

    using count_action = action_wrapper<"count"_n, &abcounter::count>;

  private:
    struct [[eosio::table]] counter {
      name key;
      uint64_t emplaced;
      uint64_t modified;
      uint64_t erased;
      uint64_t primary_key() const { return key.value; }
    };

    using count_index = eosio::multi_index<"counts"_n, counter>;
};
```

위 코드에서 처음 등장한 개념은, 아래와 코드와 같이 `require_auth()` 를 사용하여 특정 계정만이 이 컨트랙트의 어떤 액션 호출할 수 있도록 명시한다는 것입니다.

```
// addressbook 특정 계정이나 컨트랙트에게만 이 명령을 수행할 수 있는 권한을 부여한다.
require_auth( name("addressbook"));
```

앞서서는 `require_auth()` 를 사용할 때는 user 변수를 사용하였습니다.

위 코드에서 나타난 또다른 새로운 개념은 [action wrapper](https://developers.eos.io/manuals/eosio.cdt/latest/structeosio\_1\_1action\_\_wrapper) 입니다. 아래에서 볼 수 있듯이 액션 래퍼의 첫 번째 템플릿 매개변수는 우리가 호출하려는 액션이며, 두 번째는 액션 함수를 가리키는 참조자 입니다.

```cpp
using count_action = action_wrapper<"count"_n, &abcounter::count>;
```

## 단계2: abcounter 컨트랙트를 위한 계정 생성

터미널을 열고 아래 명령을 실행하여 `abcounter` 사용자를 만듭니다.

```cpp
cleos create account eosio abcounter EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

## 단계3: 컴파일 및 배포

마지막으로 `abcounter` 컨트랙트를 컴파일하고 배포합니다.

```cpp
cdt-cpp abcounter.cpp -o abcounter.wasm
```

```cpp
cleos set contract abcounter CONTRACTS_DIR/abcounter
```

## 단계4: 주소록 컨트랙트를 수정하여 인라인 액션을 abcounter 로 전송

`addressbook` 디렉토리로 이동합니다.

```cpp
cd CONTRACTS_DIR/addressbook
```

에디터로 `addressbook.cpp` 파일을 엽니다.

이전 단원의 마지막 부분에서, 자기 자신의 컨트랙트로 인라인 액션을 넘겼었습니다. 이번에는 새로운 컨트랙트인 `abcounter` 로 인라인 액션을 보낼 것입니다.

아래와 같이 `addressbook` 컨트랙트의 `private` 영역 아래에 `increment_counter` 라는 helper 메소드를 만듭니다.

```cpp
void increment_counter(name user, std::string type) {
    abcounter::count_action count("abcounter"_n, {get_self(), "active"_n});
    count.send(user, type);
}
```

위의 코드를 한번 살펴봅시다.

이번에는 함수를 호출하는 대신 `abcounter` 에서 정의한 `count_action` 액션 래퍼를 사용하고 있습니다. 이를 위해 먼저 앞서 정의한 `count_action` 객체를 초기화 했습니다. 우리가 넘긴 첫 번째 파라미터는 호출되는 컨트랙트 이름인 `abcounter` 이고, 두 번째 매개 변수는 권한 구조체입니다.

* 인증 권한에 대해 `get_self()`는 현재 `addressbook` 컨트랙트를 반환합니다. 사용되는 권한은 `addressbook` 의 active 권한입니다.

인라인 액션 추가 단원에서와는 달리 액션 래퍼 타입에 정의된 액션이 포함되므로 따로 액션을 지정할 필요가 없습니다.

세번째 줄은 `abcounter` 컨트랙트에서 요구하는 user 및 type, 즉 데이터가 포함된 액션을 호출합니다.

이제 각 액션 Scope 에 있는 helper 에 다음과 같은 호출을 추가합니다.

```cpp
//Emplace
increment_counter(user, "emplace");
//Modify
increment_counter(user, "modify");
//Erase
increment_counter(user, "erase");
```

이제 `addressbook.cpp` 전체 코드는 다음과 같을 것입니다.

```cpp
#include <eosio/eosio.hpp>
#include "abcounter.cpp"

using namespace eosio;

class [[eosio::contract("addressbook")]] addressbook : public eosio::contract {

public:

  addressbook(name receiver, name code,  datastream<const char*> ds): contract(receiver, code, ds) {}

  [[eosio::action]]
  void upsert(name user, std::string first_name, std::string last_name,
      uint64_t age, std::string street, std::string city, std::string state) {
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
      increment_counter(user, "emplace");
    }
    else {
      std::string changes;
      addresses.modify(iterator, user, [&]( auto& row ) {
        row.key = user;
        row.first_name = first_name;
        row.last_name = last_name;
        row.age = age;
        row.street = street;
        row.city = city;
        row.state = state;
      });
      send_summary(user, " successfully modified record to addressbook");
      increment_counter(user, "modify");
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
    increment_counter(user, "erase");
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

  void increment_counter(name user, std::string type) {
    abcounter::count_action count("abcounter"_n, {get_self(), "active"_n});
    count.send(user, type);
  }

  typedef eosio::multi_index<"people"_n, person,
    indexed_by<"byage"_n, const_mem_fun<person, uint64_t, &person::get_secondary_1>>
  > address_index;
};
```

## 단계5: addressbook 컨트랙트 재 컴파일 및 재배포

`addressbook.cpp` 컨트랙트를 다시 컴파일합니다. 이번에 변경된 내용은 ABI 에 영향을 주지 않기 때문에 ABI 를 다시 만들지 않습니다.&#x20;

이번엔 `-l` 옵션으로 `abcounter` 컨트랙트 폴더를 포함시킬 것입니다.

```cpp
cdt-cpp -o addressbook.wasm addressbook.cpp -I ../abcounter/
```

이제 컨트랙트를 온체인에 다시 배포합니다.

```cpp
cleos set contract addressbook CONTRACTS_DIR/addressbook
```

## 단계6: 테스트

이제 `abcounter` 가 배포되었고 `addressbook` 도 다시 배포 되었으니 테스트를 해 보겠습니다.

```cpp
$ cleos push action addressbook upsert '["alice", "alice", "liddell", 19, "123 drink me way", "wonderland", "amsterdam"]' -p alice@active

executed transaction: cc46f20da7fc431124e418ecff90aa882d9ca017a703da78477b381a0246eaf7  152 bytes  1493 us
#   addressbook <= addressbook::upsert          {"user":"alice","first_name":"alice","last_name":"liddell","street":"123 drink me way","city":"wonde...
#   addressbook <= addressbook::notify          {"user":"alice","msg":"alice successfully modified record in addressbook"}
#         alice <= addressbook::notify          {"user":"alice","msg":"alice successfully modified record in addressbook"}
#     abcounter <= abcounter::count             {"user":"alice","type":"modify"}
```

출력된 로그에서 `counter` 알림이 성공적으로 나타난 것을 확인할 수 있습니다. 이제 테이블을 확인해보겠습니다.

```cpp
$ cleos get table abcounter abcounter counts --lower alice --limit 1
{
  "rows": [{
      "key": "alice",
      "emplaced": 1,
      "modified": 0,
      "erased": 0
    }
  ],
  "more": false
}
```

각각의 액션을 테스트하고 `counter` 도 체크합니다. 이미 alice 의 레코드가 있으므로 upsert 로 수정하겠습니다.

```cpp
$ cleos push action addressbook upsert '["alice", "alice", "liddell", 21,"1 there we go", "wonderland", "amsterdam"]' -p alice@active

executed transaction: c819ffeade670e3b44a40f09cf4462384d6359b5e44dd211f4367ac6d3ccbc70  152 bytes  909 us
#   addressbook <= addressbook::upsert          {"user":"alice","first_name":"alice","last_name":"liddell","street":"1 coming down","city":"normalla...
#   addressbook <= addressbook::notify          {"user":"alice","msg":"alice successfully emplaced record to addressbook"}
>> Notified
#         alice <= addressbook::notify          {"user":"alice","msg":"alice successfully emplaced record to addressbook"}
#     abcounter <= abcounter::count             {"user":"alice","type":"emplace"}
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

alice 의 레코드는 다음과 같이 삭제할 수 있습니다.

```cpp
$ cleos push action addressbook erase '["alice"]' -p alice@active

executed transaction: aa82577cb1efecf7f2871eac062913218385f6ab2597eaf31a4c0d25ef1bd7df  104 bytes  973 us
#   addressbook <= addressbook::erase           {"user":"alice"}
>> Erased
#   addressbook <= addressbook::notify          {"user":"alice","msg":"alice successfully erased record from addressbook"}
>> Notified
#         alice <= addressbook::notify          {"user":"alice","msg":"alice successfully erased record from addressbook"}
#     abcounter <= abcounter::count             {"user":"alice","type":"erase"}
warning: transaction executed locally, but may not be confirmed by the network yet    ]
Toaster:addressbook sandwich$
```

이제 `abcounter` 컨트랙트를 직접 호출하여 데이터를 조작 할 수 있는지 테스트 해 보겠습니다.

```cpp
cleos push action abcounter count '["alice", "erase"]' -p alice@active

Error 3090004: Missing required authority
Ensure that you have the related authority inside your transaction!;
If you are currently using 'cleos push action' command, try to add the relevant authority using -p option.
Error Details:
missing authority of addressbook
pending console output:
```

권한 때문에 오류가 발생함을 확인할 수 있습니다. `abcounter` 내의 테이블을 다음과 같이 확인해 보겠습니다.

```cpp
$ cleos get table abcounter abcounter counts --lower alice

{
  "rows": [{
      "key": "alice",
      "emplaced": 1,
      "modified": 1,
      "erased": 1
    }
  ],
  "more": false
}
```

데이터 삽입, 수정, 삭제가 한번 씩 이루어졌고 그에 따라 카운터가 1 씩 증가했음을 확인할 수 있습니다. 또한 `require_auth()` 에 `name("addressbook")` 을 넣어 호출했기 때문에 `addressbook` 컨트랙트만이 이 액션을 실행할 수 있습니다. alice 가 카운팅 된 숫자들을 바꾸기 위해서 직접 `abcounter` 를 호출하면 권한 오류가 발생하고 테이블 데이터에 영향을 주지 않습니다.

## 부록: 보다 긴 영수증

아래에서 수정할 내용은 변경 모드를 기반으로 하여 사용자 영수증을 보내줄 것입니다. 만약 수정 중에 변경 사항이 없다면 영수증은 이 상황을 반영할 것입니다.

```cpp
#include <eosio/eosio.hpp>
#include "abcounter.cpp"

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
      addresses.emplace(user, [&]( auto& row ){
       row.key = user;
       row.first_name = first_name;
       row.last_name = last_name;
       row.age = age;
       row.street = street;
       row.city = city;
       row.state = state;
       send_summary(user, " successfully emplaced record to addressbook");
       increment_counter(user, "emplace");
      });
    }
    else {
      std::string changes;
      addresses.modify(iterator, user, [&]( auto& row ) {

        if(row.first_name != first_name) {
          row.first_name = first_name;
          changes += "first name ";
        }

        if(row.last_name != last_name) {
          row.last_name = last_name;
          changes += "last name ";
        }

        if(row.age != age) {
          row.age = age;
          changes += "age ";
        }

        if(row.street != street) {
          row.street = street;
          changes += "street ";
        }

        if(row.city != city) {
          row.city = city;
          changes += "city ";
        }

        if(row.state != state) {
          row.state = state;
          changes += "state ";
        }
      });

      if(!changes.empty()) {
        send_summary(user, " successfully modified record in addressbook. Fields changed: " + changes);
        increment_counter(user, "modify");
      } else {
        send_summary(user, " called upsert, but request resulted in no changes.");
      }
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
    increment_counter(user, "erase");
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

  void increment_counter(name user, std::string type) {

    action counter = action(
      permission_level{get_self(),"active"_n},
      "abcounter"_n,
      "count"_n,
      std::make_tuple(user, type)
    );

    counter.send();
  }

  typedef eosio::multi_index<"people"_n, person, indexed_by<"byage"_n, const_mem_fun<person, uint64_t, &person::get_secondary_1>>> address_index;
};
```
