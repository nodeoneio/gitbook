# 기능(Features)

## 액션의 반환값(Action Return Value)

### 개요

EOSIO 버전 2.1을 사용하면 모든 액션에서 반환값을 받을 수 있습니다. 이 기능을 사용하여 보다 쉽게 스마트 컨트랙트를 구현하고 디버깅 할 수 있으며 스마트 컨트랙트와 클라이언트 사이에서 메시지를 주고받을 수 있습니다. 이제부터 스마트 컨트랙트 클라이언트는 액션으로부터 반환된 값을 사용할 수 있기 때문에, 클라이언트 측에서 문자열을 파싱하거나 스마트 컨트랙트 구현시 Print 문을 사용할 필요가 없습니다.

### 개념

스마트 컨트랙트 내에서 액션을 작성할 때 return 문을 사용하여 어떤 값이든 액션을 호출한 쪽으로 반환 할 수 있습니다. 반환되는 값은 C++ 원시 타입, C++ 표준 라이브러리 타입 또는 사용자 정의 타입일 수 있습니다. EOSIO 프레임워크는 반환된 값을 직렬화하여 클라이언트로 다시 보내는 데 필요한 작업을 수행합니다. 클라이언트 측에서는 수신된 값을 역직렬화하여, 여타 함수의 반환 값을 사용하는것과 동일한 방법으로 사용할 수 있습니다.

### 상세

액션에서 값을 반환할때 중요한 내용이 아래에 자세히 나타나 있습니다.

* 위에서 언급한 바와 같이 EOSIO 프레임워크는 반환된 값을 클라이언트에게 전달하기 위해 모든 heavy lifting 을 수행한다. EOSIO 프레임워크는 set\_action\_return\_value 라는 새로운 내장함수를 정의하고 사용합니다. EOSIO 반환값 기능에 대한 자세한 내용은 [EOSIO 설명서 및 구현](https://github.com/EOSIO/eosio.cdt/blob/develop/libraries/native/intrinsics.cpp#L295)을 참조합니다.
* RAM 또는 NET가 아닌, 컨트랙트에 대한 CPU 타임 및 메모리 제한(wasm의 최대 크기)이 반환되는 값의 최대값을 정의합니다.
* 액션 영수증에는 직렬화된 반환 값의 해시가 포함되어 있습니다.
* 액션 추적에는 직렬화된 반환 값이 포함됩니다.
* 추적 로그가 활성화된 경우 상태 기록 추적 로그에는 직렬화된 반환 값도 저장됩니다.
* trace api 플러그인이 활성화된 경우 trace api trace 로그에 직렬화된 반환 값이 저장됩니된다.
* 반환된 값은 액션 추적에서 사용할 수 있습니다. 송신자가 다른 액션인 경우 액션 추적을 송신자의 액션 code 에서 사용할 수 없습니다. 따라서 인라인 액션에서 반환된 값을 인라인 액션을 보낸 액션에서 읽을 수 없습니다.
* 또한 인라인 액션은 동기적으로 실행되지 않고 나중에 실행됩니다. 송신자는 인라인 액션을 전송하는 시점에 반환 값을 받을 수 없습니다.

### 예제

값을 반환하는 스마트 컨트랙트 액션 예제는 다음 리소스를 참조합니다.

* Hello 스마트 컨트랙트 예제, 액션 hello::checkwithrv 을 참조.\
  [https://github.com/EOSIO/eosio.cdt/blob/develop/examples/hello/src/hello.cpp#L14](https://github.com/EOSIO/eosio.cdt/blob/develop/examples/hello/src/hello.cpp#L14)
* 액션으로부터 값을 반환하는 방법. \
  [https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/how-to-return-values-from-actions](https://developers.eos.io/manuals/eosio.cdt/latest/how-to-guides/how-to-return-values-from-actions)

## eosio::binary\_extension 타입

eosio:binary\_extension 타입이 뭔지, 무엇을 하는지, 왜 특정한 상황에서 컨트랙트 업데이트를 위해 필요한지를 설명합니다.

eosio.cdt 저장소에서 eosio::binary\_extension 을 구현한 파일인eosio.cdt/libraries/eosio/core/eosio/binary\_extension.hpp 을 찾을 수 있습니다.

이 타입을 사용할 때 가장 주목해야 할 점은, 현재의 eosio::multi\_index 타입 (즉, 테이블)에서 사용되고 있는 스마트 컨트랙트 데이터 구조에 새 필드를 추가하거나 액션 선언에 새 매개 변수를 추가할 때 입니다.

새로운 필드를 eosio::binary\_extension 으로 감싸면 나중에 사용할 수 있도록 컨트랙트가 역방향으로 호환될 수 있습니다. 이 새 필드/매개변수는 **반드시** 데이터 구조에(이는 eosio::multi\_index의 내부에 구현된 내용이 boost::multi\_index 타입에 의존하기 때문입니다) 또는 액션 선언에 있는 매개 변수 목록 끝에 추가해야 합니다.

eosio::binary\_extension 에서 새 필드를 래핑하지 않으면 eosio::multi\_index 테이블이 이전 기준점으로 읽히지 않도록 다시 포맷되거나 액션의 경우 함수를 호출할 수 없게 됩니다.

그럼 예제를 통하여 eosio::binary\_extension 타입이 작동하는 방식을 알아보겠습니다.

아래의 스마트 컨트랙트와 그에, 연결된 ABI 를 보겠습니다.

이 컨트랙트는 eosio::binary\_extension 를 설명하는 좋은 예제일 뿐 아니라, eosio 프로토콜 상의 스마트 컨트랙트 개발의 템플릿으로서 사용할 수도 있습니다.

#### binary\_extension\_contract.hpp

```cpp
#include <eosio/contract.hpp>         // eosio::contract
#include <eosio/binary_extension.hpp> // eosio::binary_extension
#include <eosio/datastream.hpp>       // eosio::datastream
#include <eosio/name.hpp>             // eosio::name
#include <eosio/multi_index.hpp>      // eosio::indexed_by, eosio::multi_index
#include <eosio/print.hpp>            // eosio::print_f

class [[eosio::contract]] binary_extension_contract : public eosio::contract {
public:
   using contract::contract;
   binary_extension_contract(eosio::name receiver, eosio::name code, eosio::datastream<const char*> ds)
      : contract{receiver, code, ds}, _table{receiver, receiver.value}
   { }

   [[eosio::action]] void regpkey (eosio::name primary_key);                ///< Register primary key.
   [[eosio::action]] void printbyp(eosio::name primary_key);                ///< Print by primary key.
   [[eosio::action]] void printbys(eosio::name secondary_key);              ///< Print by secondary key.
   [[eosio::action]] void modifyp (eosio::name primary_key, eosio::name n); ///< Modify primary key by primary key.
   [[eosio::action]] void modifys (eosio::name primary_key, eosio::name n); ///< Modify secondary key by primary key.

   struct [[eosio::table]] structure {
      eosio::name _primary_key;
      eosio::name _secondary_key;
         
      uint64_t primary_key()   const { return _primary_key.value;   }
      uint64_t secondary_key() const { return _secondary_key.value; }
   };

   using index1 = eosio::indexed_by<"index1"_n, eosio::const_mem_fun<structure, uint64_t, &structure::primary_key>>;
   using index2 = eosio::indexed_by<"index2"_n, eosio::const_mem_fun<structure, uint64_t, &structure::secondary_key>>;
   using table  = eosio::multi_index<"table"_n, structure, index1, index2>;

private:
   table _table;
};
```

#### binary\_extension\_contract.app

```cpp
#include "binary_extension_contract.hpp"

using eosio::name;

[[eosio::action]] void binary_extension_contract::regpkey(name primary_key) {
   eosio::print_f("`regpkey` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()}; ///< `index` represents `_table` organized by `index1`.
   auto iter {index.find(primary_key.value) }; ///< Note: the type returned by `index.find` is different than the type returned by `_table.find`.
   
   if (iter == _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % not found; registering.\\n", primary_key.to_string());
      _table.emplace(_self, [&](auto& row) {
         row._primary_key   = primary_key;
         row._secondary_key = "nothin"_n;
      });
   }
   else {
      eosio::print_f("`_primary_key`: % found; not registering.\\n", primary_key.to_string());
   }

   eosio::print_f("`regpkey` finished executing.\\n");
}

[[eosio::action]] void binary_extension_contract::printbyp(eosio::name primary_key) {
   eosio::print_f("`printbyp` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()};
   auto iter {index.find(primary_key.value) };
   
   if (iter != _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % found; printing.\\n", primary_key.to_string());
      eosio::print_f("{% raw %}
{%, %}
{% endraw %}\\n", iter->_primary_key, iter->_secondary_key);
   }
   else {
      eosio::print_f("`_primary_key`: % not found; not printing.\\n", primary_key.to_string());
   }

   eosio::print_f("`printbyp` finished executing.\\n");
}

[[eosio::action]] void binary_extension_contract::printbys(eosio::name secondary_key) {
   eosio::print_f("`printbys` executing.\\n");
   
   auto index{_table.get_index<"index2"_n>()};
   auto iter {index.find(secondary_key.value)};
   
   if (iter != _table.get_index<"index2"_n>().end()) {
      eosio::print_f("`_secondary_key`: % found; printing.\\n", secondary_key.to_string());
      printbyp(iter->_primary_key);
   }
   else {
      eosio::print_f("`_secondary_key`: % not found; not printing.\\n", secondary_key.to_string());
   }

   eosio::print_f("`printbys` finished executing.\\n");
}

[[eosio::action]] void binary_extension_contract::modifyp(eosio::name primary_key, name n) {
   eosio::print_f("`modifyp` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()};
   auto iter {index.find(primary_key.value)};
   
   if (iter != _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % found; modifying `_primary_key`.\\n", primary_key.to_string());
      index.modify(iter, _self, [&](auto& row) {
         row._primary_key = n;
      });
   }
   else {
      eosio::print_f("`_primary_key`: % not found; not modifying `_primary_key`.\\n", primary_key.to_string());
   }

   eosio::print_f("`modifyp` finished executing.\\n");
}

[[eosio::action]] void binary_extension_contract::modifys(eosio::name primary_key, name n) {
   eosio::print_f("`modifys` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()};
   auto iter {index.find(primary_key.value)};
   
   if (iter != _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % found; modifying `_secondary_key`.\\n", primary_key.to_string());
      index.modify(iter, _self, [&](auto& row) {
         row._secondary_key = n;
      });
   }
   else {
      eosio::print_f("`_primary_key`: % not found; not modifying `_secondary_key`.\\n", primary_key.to_string());
   }

   eosio::print_f("`modifys` finished executing.\\n");
}
```

#### binary\_extension\_contract.abi

```cpp
{
    "____comment": "This file was generated with eosio-abigen. DO NOT EDIT ",
    "version": "eosio::abi/1.1",
    "types": [],
    "structs": [
        {
            "name": "modifyp",
            "base": "",
            "fields": [
                {
                    "name": "primary_key",
                    "type": "name"
                },
                {
                    "name": "n",
                    "type": "name"
                }
            ]
        },
        {
            "name": "modifys",
            "base": "",
            "fields": [
                {
                    "name": "primary_key",
                    "type": "name"
                },
                {
                    "name": "n",
                    "type": "name"
                }
            ]
        },
        {
            "name": "printbyp",
            "base": "",
            "fields": [
                {
                    "name": "primary_key",
                    "type": "name"
                }
            ]
        },
				{
            "name": "printbys",
            "base": "",
            "fields": [
                {
                    "name": "secondary_key",
                    "type": "name"
                }
            ]
        },
        {
            "name": "regpkey",
            "base": "",
            "fields": [
                {
                    "name": "primary_key",
                    "type": "name"
                }
            ]
        },
        {
            "name": "structure",
            "base": "",
            "fields": [
                {
                    "name": "_primary_key",
                    "type": "name"
                },
                {
                    "name": "_secondary_key",
                    "type": "name"
                }
            ]
        }
    ],
		"actions": [
        {
            "name": "modifyp",
            "type": "modifyp",
            "ricardian_contract": ""
        },
        {
            "name": "modifys",
            "type": "modifys",
            "ricardian_contract": ""
        },
        {
            "name": "printbyp",
            "type": "printbyp",
            "ricardian_contract": ""
        },
        {
            "name": "printbys",
            "type": "printbys",
            "ricardian_contract": ""
        },
        {
            "name": "regpkey",
            "type": "regpkey",
            "ricardian_contract": ""
        }
    ],
    "tables": [
        {
            "name": "table",
            "type": "structure",
            "index_type": "i64",
            "key_names": [],
            "key_types": []
        }
    ],
    "ricardian_clauses": [],
    "variants": []
}
```

컨트랙트에서 업그레이드 해야 할 부분인 regpkey 액션과 structure 구조체를 아래 hpp 및 cpp 파일에 작성합니다.

#### binary\_extension\_contract.hpp

```cpp
[[eosio::action]] void regpkey (eosio::name primary_key);

struct [[eosio::table]] structure {
    eosio::name _primary_key;
    eosio::name _secondary_key;

    uint64_t primary_key()   const { return _primary_key.value;   }
    uint64_t secondary_key() const { return _secondary_key.value; }
};
```

#### binary\_extension\_contract.cpp

```cpp
[[eosio::action]] void binary_extension_contract::regpkey(name primary_key) {
   eosio::print_f("`regpkey` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()}; ///< `index` represents `_table` organized by `index1`.
   auto iter {index.find(primary_key.value) }; ///< Note: the type returned by `index.find` is different than the type returned by `_table.find`.
   
   if (iter == _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % not found; registering.\\n", primary_key.to_string());
      _table.emplace(_self, [&](auto& row) {
         row._primary_key   = primary_key;
         row._secondary_key = "nothin"_n;
      });
   }
   else {
      eosio::print_f("`_primary_key`: % found; not registering.\\n", primary_key.to_string());
   }

   eosio::print_f("`regpkey` finished executing.\\n");
}
```

ABI 파일 구조는 다음과 같습니다.

#### binary\_extension\_contract.abi

```cpp
{
    "name": "regpkey",
    "base": "",
    "fields": [
        {
            "name": "primary_key",
            "type": "name"
        }
    ]
}

{
    "name": "structure",
    "base": "",
    "fields": [
        {
            "name": "_primary_key",
            "type": "name"
        },
        {
            "name": "_secondary_key",
            "type": "name"
        }
    ]
}
```

이제 블록체인 네트워크를 시작하고 컴파일 및 테스트합니다.

```cpp
$ ~/binary_extension_contract $ cdt-cpp binary_extension_contract.cpp -o binary_extension_contract.wasm

$ ~/binary_extension_contract $ cleos set contract eosio ./

Reading WASM from /Users/john.debord/binary_extension_contract/binary_extension_contract.wasm...
Publishing contract...
executed transaction: 6c5c7d869a5be67611869b5f300bc452bc57d258d11755f12ced99c7d7fe154c  4160 bytes  729 us
#         eosio <= eosio::setcode               "0000000000ea30550000d7600061736d01000000018f011760000060017f0060027f7f0060037f7f7f017f6000017e60067...
#         eosio <= eosio::setabi                "0000000000ea3055d1020e656f73696f3a3a6162692f312e310006076d6f646966797000020b7072696d6172795f6b65790...
warning: transaction executed locally, but may not be confirmed by the network yet
```

이제 컨트랙트에 데이터를 푸시 해 보겠습니다.

```cpp
$ ~/binary_extension_contract $ cleos push action eosio regpkey '{"primary_key":"eosio.name"}' -p eosio

executed transaction: 3c708f10dcbf4412801d901eb82687e82287c2249a29a2f4e746d0116d6795f0  104 bytes  248 us
#         eosio <= eosio::regpkey               {"primary_key":"eosio.name"}
[(eosio,regpkey)->eosio]: CONSOLE OUTPUT BEGIN =====================
`regpkey` executing.
`_primary_key`: eosio.name not found; registering.
`regpkey` finished executing.
[(eosio,regpkey)->eosio]: CONSOLE OUTPUT END   =====================
warning: transaction executed locally, but may not be confirmed by the network yet
```

마지막으로 작성한 데이터를 다시 읽어보겠습니다.

```cpp
$ ~/binary_extension_contract $ cleos push action eosio printbyp '{"primary_key":"eosio.name"}' -p eosio

executed transaction: e9b77d3cfba322a7a3a93970c0c883cb8b67e2072a26d714d46eef9d79b2f55e  104 bytes  227 us
#         eosio <= eosio::printbyp              {"primary_key":"eosio.name"}
[(eosio,printbyp)->eosio]: CONSOLE OUTPUT BEGIN =====================
`printbyp` executing.
`_primary_key`: eosio.name found; printing.
{eosio.name, nothin}
`printbyp` finished executing.
[(eosio,printbyp)->eosio]: CONSOLE OUTPUT END   =====================
warning: transaction executed locally, but may not be confirmed by the network yet
```

이제 테이블에 새로운 필드를 더하고 액션에 새로운 매개변수를 더하여 스마트 컨트랙트를 업그레이드 해 보겠습니다. 이 때 eosio::binary\_extension 로 새로운 필드와 매개변수를 래핑하지는 않을 것인데, 그러면 무슨 일이 일어나나 확인해 볼 것입니다.

#### binary\_extension\_contract.hpp

```cpp
+[[eosio::action]] void regpkey (eosio::name primary_key, eosio::name secondary_key);
-[[eosio::action]] void regpkey (eosio::name primary_key);

struct [[eosio::table]] structure {
    eosio::name _primary_key;
    eosio::name _secondary_key;
+   eosio::name _non_binary_extension_key;

    uint64_t primary_key()   const { return _primary_key.value;   }
    uint64_t secondary_key() const { return _secondary_key.value; }
};
```

#### binary\_extension\_contract.cpp

```cpp
+[[eosio::action]] void binary_extension_contract::regpkey(name primary_key, name secondary_key) {
-[[eosio::action]] void binary_extension_contract::regpkey(name primary_key) {
   eosio::print_f("`regpkey` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()}; ///< `index` represents `_table` organized by `index1`.
   auto iter {index.find(primary_key.value) }; ///< Note: the type returned by `index.find` is different than the type returned by `_table.find`.
   
   if (iter == _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % not found; registering.\\n", primary_key.to_string());
      _table.emplace(_self, [&](auto& row) {
         row._primary_key   = primary_key;
+        if (secondary_key) {
+           row._secondary_key = secondary_key;
+         }
+         else {
            row._secondary_key = "nothin"_n;
+         }
      });
   }
   else {
      eosio::print_f("`_primary_key`: % found; not registering.\\n", primary_key.to_string());
   }

   eosio::print_f("`regpkey` finished executing.\\n");
}
```

#### binary\_extension\_contract.abi

```cpp
{
    "name": "regpkey",
    "base": "",
    "fields": [
        {
            "name": "primary_key",
            "type": "name"
+       },
+       {
+           "name": "secondary_key",
+           "type": "name"
        }
    ]
}
{
    "name": "structure",
    "base": "",
    "fields": [
        {
            "name": "_primary_key",
            "type": "name"
        },
        {
            "name": "_secondary_key",
            "type": "name"
+       },
+	{
+           "name": "_non_binary_extension_key",
+           "type": "name"
        }
    ]
}
```

이제 컨트랙트를 업그레이드 하고 원래 방식대로 테이블에서 읽고 쓰기를 해 보겠습니다.

```cpp
$ ~/binary_extension_contract $ cdt-cpp binary_extension_contract.cpp -o binary_extension_contract.wasm

$ ~/binary_extension_contract $ cleos set contract eosio ./

Reading WASM from /Users/john.debord/binary_extension_contract/binary_extension_contract.wasm...
Publishing contract...
executed transaction: b8ea485842fa5645e61d35edd97e78858e062409efcd0a4099d69385d9bc6b3e  4408 bytes  664 us
#         eosio <= eosio::setcode               "0000000000ea30550000a2660061736d01000000018f011760000060017f0060027f7f0060037f7f7f017f6000017e60067...
#         eosio <= eosio::setabi                "0000000000ea305583030e656f73696f3a3a6162692f312e310006076d6f646966797000020b7072696d6172795f6b65790...
warning: transaction executed locally, but may not be confirmed by the network yet

$ ~/binary_extension_contract $ cleos push action eosio printbyp '{"primary_key":"eosio.name"}' -p eosio

Error 3050003: eosio_assert_message assertion failure
Error Details:
assertion failure with message: read
```

앞서 테이블에 삽입한 데이터를 읽을 수 없는 것을 확인할 수 있습니다.

```cpp
$ ~/binary_extension_contract $ cleos push action eosio regpkey '{"primary_key":"eosio.name2"}' -p eosio

Error 3015014: Pack data exception
Error Details:
Missing field 'secondary_key' in input object while processing struct 'regpkey'
```

마찬가지로 업그레이드 된 액션으로도 원래 방식대로는 테이블에 쓸 수 없습니다.

이제 eosio::binary\_extension 타입으로 새로운 필드와 매개변수를 래핑 해보겠습니다.

#### binary\_extension\_contract.hpp

```cpp
+[[eosio::action]] void regpkey (eosio::name primary_key. eosio::binary_extension<eosio::name> secondary_key);
-[[eosio::action]] void regpkey (eosio::name primary_key, eosio::name secondary_key);

struct [[eosio::table]] structure {
    eosio::name                          _primary_key;
    eosio::name                          _secondary_key;
+   eosio::binary_extension<eosio::name> _binary_extension_key;
-   eosio::name                          _non_binary_extension_key;

    uint64_t primary_key()   const { return _primary_key.value;   }
    uint64_t secondary_key() const { return _secondary_key.value; }
};
```

#### binary\_extension\_contraact.cpp

```cpp
+[[eosio::action]] void binary_extension_contract::regpkey(name primary_key, binary_extension<name> secondary_key) {
-[[eosio::action]] void binary_extension_contract::regpkey(name primary_key, name secondary_key) {
   eosio::print_f("`regpkey` executing.\\n");
   
   auto index{_table.get_index<"index1"_n>()}; ///< `index` represents `_table` organized by `index1`.
   auto iter {index.find(primary_key.value) }; ///< Note: the type returned by `index.find` is different than the type returned by `_table.find`.
   
   if (iter == _table.get_index<"index1"_n>().end()) {
      eosio::print_f("`_primary_key`: % not found; registering.\\n", primary_key.to_string());
      _table.emplace(_self, [&](auto& row) {
         row._primary_key   = primary_key;
         if (secondary_key) {
+           row._secondary_key = secondary_key.value();
-           row._secondary_key = secondary_key;
          }
          else {
            row._secondary_key = "nothin"_n;
          }
      });
   }
   else {
      eosio::print_f("`_primary_key`: % found; not registering.\\n", primary_key.to_string());
   }

   eosio::print_f("`regpkey` finished executing.\\n");
}
```

#### binary\_extension\_contract.abi

```cpp
{
    "name": "regpkey",
    "base": "",
    "fields": [
        {
            "name": "primary_key",
            "type": "name"
        },
        {
            "name": "secondary_key",
+           "type": "name$"
-           "type": "name"
        }
    ]
}
{
    "name": "structure",
    "base": "",
    "fields": [
        {
            "name": "_primary_key",
            "type": "name"
        },
        {
            "name": "_secondary_key",
            "type": "name"
        },
	{
+           "name": "_binary_extension_key",
+           "type": "name$"
-           "name": "_non_binary_extension_key",
-           "type": "name"
        }
    ]
}
```

이번엔 타입 뒤에 "$" 가 붙어 있는 것을 볼 수 있습니다. 이것이 eosio::binary\_extension 타입 필드라는 것을 나타내는 것입니이다.

```cpp
{
    "name": "secondary_key",
+   "type": "name$"
-   "type": "name"
}
{
    "name": "_binary_extension_key",
+   "type": "name$"
-   "type": "name"
}
```

이제 컨트랙트를 업그레이드 하고 테이블에 읽기/쓰기를 해 보겠습니다.

```cpp
$ ~/binary_extension_contract $ cleos set contract eosio ./

Reading WASM from /Users/john.debord/binary_extension_contract/binary_extension_contract.wasm...
Publishing contract...
executed transaction: 497584d4e43ec114dbef83c134570492893f49eacb555d0cd47d08ea4a3a72f7  4696 bytes  648 us
#         eosio <= eosio::setcode               "0000000000ea30550000cb6a0061736d01000000018f011760000060017f0060027f7f0060037f7f7f017f6000017e60017...
#         eosio <= eosio::setabi                "0000000000ea305581030e656f73696f3a3a6162692f312e310006076d6f646966797000020b7072696d6172795f6b65790...
warning: transaction executed locally, but may not be confirmed by the network yet

$~/binary_extension_contract $ cleos push action eosio printbyp '{"primary_key":"eosio.name"}' -p eosio

executed transaction: 6108f3206e1824fe3a1fdcbc2fe733f38dc07ae3d411a1ccf777ecef56ddec97  104 bytes  224 us
#         eosio <= eosio::printbyp              {"primary_key":"eosio.name"}
[(eosio,printbyp)->eosio]: CONSOLE OUTPUT BEGIN =====================
`printbyp` executing.
`_primary_key`: eosio.name found; printing.
{eosio.name, nothin}
`printbyp` finished executing.
[(eosio,printbyp)->eosio]: CONSOLE OUTPUT END   =====================
warning: transaction executed locally, but may not be confirmed by the network yet

$~/binary_extension_contract $ cleos push action eosio regpkey '{"primary_key":"eosio.name2"}' -p eosio

executed transaction: 75a135d1279a9c967078b0ebe337dc0cd58e1ccd07e370a899d9769391509afc  104 bytes  227 us
#         eosio <= eosio::regpkey               {"primary_key":"eosio.name2"}
[(eosio,regpkey)->eosio]: CONSOLE OUTPUT BEGIN =====================
`regpkey` executing.
`_primary_key`: eosio.name2 not found; registering.
`regpkey` finished executing.
[(eosio,regpkey)->eosio]: CONSOLE OUTPUT END   =====================
warning: transaction executed locally, but may not be confirmed by the network yet
```

잘 실행됩니다. 이제 이 스마트 컨트랙트는 나중에 테이블/액션을 사용할 때를 위한 역방향(backward) 호환성을 가지게 되었습니다.

스마트 컨트랙트를 업그레이드 할 때 다음 규칙을 기억합시다. eosio:multi\_index 를 사용하여 새로운 필드를 구조체에 추가할 때는, 새 필드를 구조체의 가장 마지막에 추가해야 하는할 것과 eosio::binary\_extension 타입을 사용하여 래핑해야 한다는 것을 유의해야 합니다.

## 네이티브 테스터와 컴파일

v1.5.0 이후로 네이티브 컴파일을 실행할 수 있으며, 이를 위하여 네이티브 테스팅 및 네이티브 "스크래치 패드(Scratch Pad)" 컴파일을 지원하는 새로운 라이브러리 세트가 준비되어 있습니다. 이제 eosio-cc, cdt-cpp 및 eosio-ld는 더 편하고 빠른 개발을 위한 네이티브 유닛 테스트와 "스마트 컨트랙트" 구축을 기본적으로 지원합니다.(현재 eosio 내장함수(intrinsics) 의 기본 구현은 사용할 수 없는 상태이며 사용자가 정의할 수 있는 상태임을 참고).(note the default implementations of eosio intrinsics are currently asserts that state they are unavailable, these are user definable.)

## 시작

일단 스마트 컨트랙트를 작성하고면 테스트 소스도 작성합니다.

hello.hpp

```cpp
#include <eosio/eosio.hpp>

using namespace eosio;

class [[eosio::contract]] hello : public eosio::contract {
  public:
      using contract::contract;

      [[eosio::action]] void hi( name user );

      // 외부 컨트랙트가 이 컨트랙트로 쉽게 인라인 액션을 보낼 수 있도록 하는 접근자
      using hi_action = action_wrapper<"hi"_n, &hello::hi>;
};
```

그리고 hello\_test.cpp 테스트 코드를 작성합니다.

```cpp
#include <eosio/eosio.hpp>
#include <eosio/tester.hpp>

#include <hello.hpp>

using namespace eosio;
using namespace eosio::native;

EOSIO_TEST_BEGIN(hello_test)
   // These can be redefined by the user to suit there needs per unit test
   // the idea is that in a future release there will be a base library that 
   // initializes these to "useable" default implementations and probably 
   // helpers to more easily define read_action_data and action_data_size intrinsics
   // like these"
   intrinsics::set_intrinsic<intrinsics::read_action_data>(
         [](void* m, uint32_t len) {
            check(len <= sizeof(eosio::name), "failed from read_action_data");
            *((eosio::name*)m) = "hello"_n;
            return len; 
         });

   intrinsics::set_intrinsic<intrinsics::action_data_size>(
         []() {
            return (uint32_t)sizeof(eosio::name);
         });
   
   intrinsics::set_intrinsic<intrinsics::require_auth>(
         [](capi_name nm) {
         });

   
   // "Name : hello" should be in the print buffer
   CHECK_PRINT("Name : hello",
         []() {
            apply("test"_n.value, "test"_n.value, "hi"_n.value);
            });
           
   // should not assert
   apply("test"_n.value, "test"_n.value, "check"_n.value);
   
   name nm = "null"_n;
   intrinsics::set_intrinsic<intrinsics::read_action_data>(
         [&](void* m, uint32_t len) {
            check(len <= sizeof(eosio::name), "failed from read_action_data");
            *((eosio::name*)m) = nm;
            return len; 
         });

   REQUIRE_ASSERT( "check name not equal to `hello`",
         []() {
            // should assert
            apply("test"_n.value, "test"_n.value, "check"_n.value);
            });

EOSIO_TEST_END

// boilerplate main, this will be generated in a future release
int main(int argc, char** argv) {
   silence_output(true);
   EOSIO_TEST(hello_test);
   return has_failed();
}
```

eosio 에 정의된 모든 내장함수(prints, require\_auth 등) 들은  `intrinsics::set_intrinsics<intrinsics::the_intrinsic_name>()` 함수로 재정의 할 수 있습니다. 이러한 함수들은 람다 함수를 가지는데 이 람다 함수들은 인수와 리턴 타입이 재정의하고자 하는 내장 함수와 일치해야 합니다. 이러한 특성덕분에 컨트랙트 작성자는 작성중인 유닛 테스트를 유연하게 수정하여 원하는대로 작동하게 할 수 있습니다.

다른 함수인 `intrinsics::get_intrinsics<intrinsics::the_intrinsic_name>()` 은 현재 내장함수의 행동을 정의하는 함수 개체를 반환합니다. 이 패턴은 mock 함수에 사용될 수 있으며 스마트 컨트랙트를 쉽게 테스트 할 수 있게 합니다. 다음 링크에서 더 많은 정보를 확인할 수 있습니다.

[https://github.com/EOSIO/eosio.cdt/blob/master/examples/hello/tests/](https://github.com/EOSIO/eosio.cdt/blob/master/examples/hello/tests/)

[https://github.com/EOSIO/eosio.cdt/blob/master/examples/hello/tests/hello\_test.cpp](https://github.com/EOSIO/eosio.cdt/blob/master/examples/hello/tests/hello\_test.cpp)

## 네이티브 코드 컴파일

* 테스트 또는 프로그램을 만들기 위해 `cdt-cpp` 로 파일을 컴파일 할 때, 추가적으로 명령줄에`-fnative` 플래그를 추가합니다. 이렇게 하면 wasm 코드 대신 네이티브 코드가 생성됩니다.
* CMake 를 사용
  * `add_native_library` 와 `add_native_executable` CMake 매크로가 추가되어 있습니다. 이것들은 add\_library 와 add\_executable 을 대체합니다.

### Eosio.CDT 네이티브 테스터 API

* CHECK\_ASSERT(...): 이 매크로는 특정 Assert 가 발생했는지 체크하고 테스트를 실패로 플래그 합니다. 그리고 나머지 테스트들을 계속 수행합니다.
  * 다음 두 가지 방식으로 호출 할 수 있습니다.
    * `CHECK_ASSERT("<assert message>", [](<args>){ whatever_function(<args>); })`
    * `CHECK_ASSERT([](std::string msg){ user defined comparison function }, [](<args>){ whatever_function(<args>); })`
* CHECK\_PRINT(...): 이 매크로는 프린트 버퍼가 특정한 문자열을 가지고 있는지 체크하고 테스트를 실패로 플래그합니다. 그리고 나머지 테스트들을 수행합니다.
  * 다음 두 가지 방식으로 호출할 수 있습니다.
    * `CHECK_PRINT("<print message>", [](<args>){ whatever_function(<args>); })`
    * `CHECK_PRINT([](std::string print_buffer){ user defined comparison function }, [](<args>){ whatever_function(<args>); })`
* CHECK\_EQUAL(X,Y): 이 매크로는 두 입력값이 같은지 확인하고 테스트를 실패로 플래그 합니다. 그리고 나머지 테스트들을 수행합니다.
* REQUIRE\_ASSERT(...): 이 매크로는 특정한 ASSERT 가 일어났는지 체크하고 테스트를 실패로 플래그 합니다. 그리고 전체 테스트를 중지한다.
  * 다음 두 가지 방식으로 호출할 수 있다.
    * `REQUIRE_ASSERT("<assert message>", [](<args>){ whatever_function(<args>); })`
    * `REQUIRE_ASSERT([](std::string msg){ user defined comparison function }, [](<args>){ whatever_function(<args>); })`
* REQUIRE\_PRINT(...): 이 매크로는 프린트 버퍼가 특정한 문자열을 가지고 있는지 체크하고 테스트를 실패로 플래그한다. 그리고 전체 테스트를 중지합니다.
  * 다음 두 가지 방식으로 호출할 수 있다.
    * `REQUIRE_PRINT("<print message>", [](<args>){ whatever_function(<args>); })`
    * `REQUIRE_PRINT([](std::string print_buffer){ user defined comparison function }, [](<args>){ whatever_function(<args>); })`
* REQUIRE\_EQUAL(X,Y): 이 매크로는 X, Y 두 입력값이 서로 일치하는지 체크하고 테스트를 실패로 플래그한 뒤 전체 테스트를 중지합니다.
* EOSIO\_TEST\_BEGIN(X): 이 매크로는 유닛 테스트의 시작을 정의하고 X를 테스트의 심볼릭 네임으로 지정합니다.
* EOSIO\_TEST\_END: 이 매크로는 유닛 테스트의 종료를 정의합니다.
* EOSIO\_TEST(X): 이 매크로는 메인 함수에서 X라는 이름의 테스트를 실행하는데 사용합니다.

## 자원 지불자(Resource Payer)

{% hint style="warning" %}
Resource Payer 는 EOSIO 2.2의 기능이므로 만델에서 적용되는지 여부를 보고 번역한다.

[https://developers.eos.io/manuals/eosio.cdt/latest/features/resource\_payer](https://developers.eos.io/manuals/eosio.cdt/latest/features/resource\_payer)
{% endhint %}

