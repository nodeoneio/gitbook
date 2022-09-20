# 사용자 지정 권한(Custom permission) 연결: 스마트 컨트랙트에서 사용자 지정 권한 설정

Antelope 의 권한은 Antelope 블록체인에 대해 계정이 실행할 수 있는 권한을 제어합니다. Antelope 의 권한은 강력하고 유연하며 사용자가 직접 정의할 수도 있습니다. 이 단원에서는 사용자 지정 권한과 그것을 사용하는 방법을 설명할 것입니다. 그리고 사용자 지정 권한을 생성하고, 사용 권한을 액션에 연결하며, 액션에서 사용 권한을 해제하고, 사용자 지정 권한을 삭제하는 방법을 실습할 것입니다.

## 시작하기 전에

이 단원은 다음과 같은 내용이 필요합니다.

* 실행 중이며 접근 가능한 블록체인. 블록 체인 실행에 대한 지침을 보려면 이 링크를 클릭합니다.
* Antelope 계정 및 계정 키에 대한 액세스 권한. 계정 및 사용 권한에 대한 정보를 보려면 이 링크를 클릭합니다.
* 스마트 컨트랙트에 대한 기본 지식. [Hello World 튜토리얼](../smart-contracts-dev-environment/contract-dev-workflow/hello-world-contract.md)을 참조합니다.

### 사용자 지정 권한이란?

사용자 지정 권한은 Antelope 계정과 연결되어 임의로 이름붙여진 권한입니다. 계정이 생성되면 기본적으로 `owner` 와 `active` 두 가지 권한이 부여됩니다. `owner`, `active` 또는 다른 사용자 지정 권한의 하위 권한으로 새로운 사용자 지정 권한을 만들 수 있습니다. 사용자 지정 권한은 공개 키 / 개인 키 쌍을 필요로 합니다. 사용자 지정 권한은 스마트 컨트랙트의 액션에 연결되어 해당 액션을 실행하는 데 필요한 권한을 지정합니다. Antelope 계정 및 사용 권한을 통해 계정 및 스마트 컨트랙트 액션을 유연하고 세밀하게 제어할 수 있습니다.

{% hint style="info" %}
`eosio.code` 권한은 스마트 컨트랙트에서 인라인 액션을 호출할 수 있는 특수 권한입니다. Antelope 블록체인은 스마트 컨트랙트의 인라인 액션이 호출될 때 적절한 권한이 부여되어 있는지 확인합니다.&#x20;

원래 스마트 컨트랙트에는 계정 키에 대한 액세스 권한이 없으므로 권한을 추가할 수 없으나, 계정 권한에 `eosio.code` 를 추가하는 것으로 해당 계정 권한이 인라인 액션을 실행할 수 있는 명시적 권한을 제공할 수 있게 됩니다.

`eosio.code` 추가 방법에 대한 자세한 내용은 `cleos set account permission` 을 참조합니다.
{% endhint %}

### 사용자 지정 권한을 사용하는 이유

사용자 지정 권한을 사용하면 사용자의 계정을 사용할 때와 특정한 트랜잭션을 호출 할 때 보안을 강화할 수 있습니다. 또한 사용자의 `owner` 키와 active 키의 사용 빈도를 줄일 수 있고, 덕분에 `owner` 와 `active` 권한이 노출되거나 손상될 위험을 방지할 수 있습니다.

{% hint style="info" %}
어떤 사용자 지정 권한에는 항상 상위 권한이 있습니다. 상위 권한은 사용자 지정 권한과 동일한 트랜잭션을 실행할 권한을 가집니다. 상위 권한은 하위의 사용자 지정 권한이 손상된 경우 그 사용자 지정 권한 키를 복구할 수 있는 권한도 가집니다.
{% endhint %}

### 사용자 지정 권한 사용하기

이제부터 무엇을 할 것이며 무엇이 필요한지 자세히 설명하고, 테스트에 사용할 수 있는 간단한 스마트 컨트랙트를 만들어 보겠습니다.

다음 내용을 실습해 보겠습니다.

* 사용자 지정 권한 만들기
* 사용자 지정 권한 연결 하기
* 사용자 지정 권한 연결 끊기
* 사용자 지정 권한 삭제하기

이를 위하여 다음 내용들이 필요합니다.

* `bob` 이라는 계정과 이 계정을 제어할 수 있는 키가 로컬 지갑에 저장되어 있어야 합니다.
* `scholder` 라는 계정과 이 계정을 제어할 수 있는 키가 로컬 지갑에 저장되어 있어야 합니다.
* `scholder` 계정에 배포된 `hello` 라는 이름의 스마트 컨트랙트. 이 컨트랙트는 다음과 같은 3개의 액션을 가지고 있습니다.
  * `what(eosio::name user)`
  * `why(eosio::name user)`
  * `how(eosio::name user)`

이제 계정 `bob` 으로 2개의 사용자 지정 권한을 만들어보겠습니다.

* active 를 상위 권한으로 가지는 사용자 지정 권한 customp1.
* customp1 를 상위 권한으로 가지는 사용자 지정 권한 customp2.

이 사용자 지정 권한들을 다음과 같이 스마트 컨트랙트 액션과 연결하겠습니다.

* customp1 을 what(eosio::name user) 에 연결
* customp2 을 how(eosio::name user) 에 연결

active 권한은 모든 액션을 호출할 수 있습니다. customp1 권한은 why 와 how 액션을 호출할 수 있으며, customp2 권한은 how 액션만을 호출할 수 있습니다.

그 다음 customp2 권한의 연결을 끊고 삭제할 수 있습니다.

### 간단한 스마트 컨트랙트

본 예제에서 사용할 스마트 컨트랙트는 다음과 같습니다.

```cpp
#include <eosio/eosio.hpp>
class [[eosio::contract]] hello : public eosio::contract {
  public:
      using eosio::contract::contract;
      [[eosio::action]] void what( eosio::name user ) {
         print( "hi, what do you want ", user);
      }

      [[eosio::action]] void why( eosio::name user ) {
         print( "why not ", user);
      }

      [[eosio::action]] void how( eosio::name user ) {
         print( "how are you ", user);
      }
};
```

위 컨트랙트를 실행중인 블록체인에 배포합니다.

{% hint style="info" %}
이 단원은 네이티브 권한 체크를 시연하려는 것이기 때문에 위 스마트 컨트랙트 코드는 `require_auth()` 로 권한 체크를 하지 않습니다. 컨트랙트 보안 정보는 아래 링크를 참조합니다. https://developers.eos.io/manuals/eosio.cdt/latest/best-practices/securing\_your\_contract/#1-authorization-checks
{% endhint %}

### 사용자 지정 권한 만들기

이제 사용자 지정 권한을 만들고 사용해 봅시다.

아래 `cleos set account permission` 명령을 사용하면 bob 의 계정에 `active` 를 상위 권한으로 가지는 사용자 지정 권한 customp1 을 만들 수 있습니다.

```cpp
$ cleos set account permission bob customp1 EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D active -p bob@active

executed transaction: 97e2af6966b40ea0b523402110c6a5592862c5ad2abbaad20c9bbf2f68017c98  160 bytes  145 us
#         eosio <= eosio::updateauth            {"account":"bob","permission":"customp1","parent":"active","auth":{"threshold":1,"keys":[{"key":"EOS...
```

이제 다시 bob 계정에 customp1 을 상위 권한으로 갖는 사용자 지정 권한 customp2 를 만듭니다.

```cpp
$ cleos set account permission bob customp2 EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D customp1 -p bob@active

executed transaction: 8b5e88d0d1cea6dd31a4967912d575d62391348345c58b6071aba7fb93d709b3  160 bytes  129 us
#         eosio <= eosio::updateauth            {"account":"bob","permission":"customp2","parent":"customp1","auth":{"threshold":1,"keys":[{"key":"E...
```

또한 상위 권한을 지정하지 않고 사용자 지정 권한을 만들 수도 있는데 이 경우 디폴트로 `active` 가 상위 권한이 됩니다.

```cpp
$ cleos set account permission bob customp3 EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D -p bob@active

executed transaction: 42158f0a35bdce9e8374691285e12e5e517ab4d4831a68c8e3a2bb22e88fda7c  160 bytes  165 us
#         eosio <= eosio::updateauth            {"account":"bob","permission":"customp3","parent":"active","auth":{"threshold":1,"keys":[{"key":"EOS...
```

이제 계정을 확인해 봅시다.

```cpp
$ cleos get account bob --json

{
  "account_name": "bob",
  "head_block_num": 23485,
  "head_block_time": "2021-05-06T07:46:19.000",
  "privileged": false,
  "last_code_update": "1970-01-01T00:00:00.000",
  "created": "2021-05-06T05:07:27.500",
  "ram_quota": -1,
  "net_weight": -1,
  "cpu_weight": -1,
  "net_limit": {
    "used": -1,
    "available": -1,
    "max": -1
  },
  "cpu_limit": {
    "used": -1,
    "available": -1,
    "max": -1
  },
  "ram_usage": 4414,
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "customp1",
      "parent": "active",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "customp2",
      "parent": "customp1",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },,{
      "perm_name": "customp3",
      "parent": "active",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    }
  ],
  "total_resources": null,
  "self_delegated_bandwidth": null,
  "refund_request": null,
  "voter_info": null,
  "rex_info": null
}
```

{% hint style="info" %}
내용을 단순하게 진행하기 위해 각 권한의 공개키/개인키를 모두 동일한 것으로 하였습니다. 실제 운영 환경에서는 각 권한마다 새로운 키를 만들어 사용해야 합니다.
{% endhint %}

### 권한 연결하기

사용자 지정 권한을 만들고 나면 이 권한을 스마트 컨트랙트 액션에 연결할 수 있습니다. 이 액션을 실행하려면 사용자 권한이 이 액션이 필요로 하는 권한 또는 그보다 상위 권한을 가지고 있어야 합니다.

`cleos set action permission` 명령을 사용하여 customp1 권한을 what 액션에 연결해 보겠습니다.

```cpp
$ cleos set action permission bob scholder what customp1 -p bob@active

executed transaction: 64198d1cc5f7dedf8809b86f22801eb004d50365ba72f8e2833ed191c6f6e30b  128 bytes  471 us
#         eosio <= eosio::linkauth              {"account":"bob","code":"scholder","type":"what","requirement":"customp1"}
```

마찬가지로 cleos set action permission 명령으로 customp2 권한을 how 액션에 연결해 보겠습니다.

```cpp
$ cleos set action permission bob scholder how customp2 -p bob@active

executed transaction: 64198d1cc5f7dedf8809b86f22801eb004d50365ba72f8e2833ed191c6f6e30b  128 bytes  471 us
#         eosio <= eosio::linkauth              {"account":"bob","code":"scholder","type":"how","requirement":"customp2"}
```

### 테스트

이제 what 과 how 액션에 연결된 두 개의 사용자 지정 권한 customp1, customp2 를 만들었습니다. 각 권한과 액션과의 관계를 정리하면 다음과 같습니다.

* active 은 why, what, how 를 호출할 수 있습니다. active 권한은 customp1 의 상위 권한이므로 customp1 이 호출할 수 있는 모든 액션을 호출 할 수 있습니다.
* customp1 은 what, how 를 호출할 수 있습니다다. customp1 권한은 customp2 의 상위 권한이므로 customp2 이 호출할 수 있는 모든 액션을 호출 할 수 있습니다.
* customp2 는 how 를 호출할 수 있습니다.

이제 권한을 사용하여 스마트 컨트랙트 액션을 호출하는 방식으로 테스트를 해 보겠습니다.

#### active 권한으로 액션 호출하기

bob@active 로 why 를 호출 해 봅시다.

```cpp
$ cleos push action scholder why '["name"]' -p bob@active

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::why               {"user":"name"}
>> why not name
```

bob@active 로 what 를 호출 해 보겠습니다.

```cpp
$ cleos push action scholder what '["name"]' -p bob@active

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::what               {"user":"name"}
>> hi, what do you want name
```

bob@active 로 how 를 호출 해 보겠습니다.

```cpp
$ cleos push action scholder how '["name"]' -p bob@active

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::how               {"user":"name"}
>> how are you name
```

active 권한이 모든 액션을 호출할 수 있음을 확인하였습니다.

#### customp1 권한으로 액션 호출하기

bob@customp1 로 why 를 호출 해 보겠습니다.

```cpp
$ cleos push action scholder why '["name"]' -p bob@customp1

Error 3090005: Irrelevant authority included
Please remove the unnecessary authority from your action!
Error Details:
action declares irrelevant authority '{"actor":"bob","permission":"customp1"}'; minimum authority is {"actor":"bob","permission":"active"}
```

권한 오류가 발생하였습니다. 이번엔 bob@customp1 로 what 을 호출 해 보겠습니다.

```cpp
$ cleos push action scholder what '["name"]' -p bob@customp1

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::what               {"user":"name"}
>> hi, what do you want name
```

bob@customp1 로 how 를 호출 해 보겠습니다.

```cpp
$ cleos push action scholder how '["name"]' -p bob@customp1

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::how               {"user":"name"}
>> how are you name
```

customp1 권한은 what과 how 를 호출 할 수 있지만 why 는 호출할 수 없다는 것을 확인하였습니다.

#### customp2 권한으로 액션 호출하기

bob@customp2 로 why 를 호출 해 보겠습니다.

```cpp
$ cleos push action scholder why '["name"]' -p bob@customp2

Error 3090005: Irrelevant authority included
Please remove the unnecessary authority from your action!
Error Details:
action declares irrelevant authority '{"actor":"bob","permission":"customp1"}'; minimum authority is {"actor":"bob","permission":"active"}
```

권한 오류가 발생했습니다. bob@customp2 로 what 을 호출 해 보겠습니다.

```cpp
$ cleos push action scholder what '["name"]' -p bob@customp2

Error 3090005: Irrelevant authority included
Please remove the unnecessary authority from your action!
Error Details:
action declares irrelevant authority '{"actor":"bob","permission":"customp1"}'; minimum authority is {"actor":"bob","permission":"active"}
```

마찬가지로 권한 오류가 발생합니다. bob@customp2 로 how 를 호출 해 보겠습니다.

```cpp
$ cleos push action scholder how '["name"]' -p bob@customp2

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::how               {"user":"name"}
>> how are you name
```

customp2 권한은 how 를 호출 할 수 있지만 what 과 why 는 호출할 수 없음을 확인하였습니다.

### 권한 연결 끊기

이제 스마트 컨트랙트 액션과 권한의 연결을 끊어보겠습니다.

`cleos set action permission` 명령을 사용하여 how 액션과 customp2 권한의 연결을 끊겠습니다.

```cpp
$ cleos set action permission bob scholder how NULL -p bob@active

executed transaction: 50fe754760a1b8bd0e56f57570290a3f5daa509c090deb54c81a721ee7048201  120 bytes  242 us
#         eosio <= eosio::unlinkauth            {"account":"bob","code":"scholder","type":"how"}
```

### 테스트

이제 customp2 권한은 연결이 끊어졌습니다.

이제 what 액션을 호출할 수 있는 customp1 권한만이 남아 있습니다. 그리고 customp2 권한은 아무것도 호출 할 수 없게 되었습니다. 정리하면 다음과 같습니다.

* active 는 why, what, how 를 호출할 수 있습니다. active 권한은 customp1 권한의 상위 권한이므로 customp1 권한이 호출할 수 있는 모든 액션을 호출할 수 있습니다.
* customp1 은 what 을 호출할 수 있습니다.
* customp2 은 연결된 액션이 없기 때문에 아무것도 호출 할 수 없습니다.

#### 이 단원에서 만든 권한들로 how 액션을 호출해 본 뒤, 어떤 권한이 이 액션을 호출할 수 있는지 확인합니다.

bob@active 로 how 를 호출해 봅니다.

```cpp
$ cleos push action scholder how '["name"]' -p bob@active

executed transaction: 43b6ad4ce7a52d7281ccd2800caa02d5278ee714de36fefe9624bff621378402  104 bytes  129 us
#      scholder <= scholder::how               {"user":"name"}
>> how are you name
```

bob@customp2 로 how 를 호출해 봅니다.

```cpp
$ cleos push action scholder how '["name"]' -p bob@customp2

Error 3090005: Irrelevant authority included
Please remove the unnecessary authority from your action!
Error Details:
action declares irrelevant authority '{"actor":"bob","permission":"customp2"}'; minimum authority is {"actor":"bob","permission":"active"}
```

### 사용자 지정 권한 삭제

`cleos set account permission` 명령을 사용하여 모든 액션과의 연결이 끊긴 customp2 권한을 삭제할 수 있습니다.

```cpp
$ cleos set account permission bob customp2 NULL active -p bob@active

executed transaction: 3f3e58707e5548ec34f5655327b1110c18d455c9ee0a6cffc102d7bc4e0a6cdb  112 bytes  472 us
#         eosio <= eosio::deleteauth            {"account":"bob","permission":"customp2"}
```

이제 계정을 확인해 보겠습니다

```cpp
$ cleos get account bob --json

{
  "account_name": "bob",
  "head_block_num": 23485,
  "head_block_time": "2021-05-06T07:46:19.000",
  "privileged": false,
  "last_code_update": "1970-01-01T00:00:00.000",
  "created": "2021-05-06T05:07:27.500",
  "ram_quota": -1,
  "net_weight": -1,
  "cpu_weight": -1,
  "net_limit": {
    "used": -1,
    "available": -1,
    "max": -1
  },
  "cpu_limit": {
    "used": -1,
    "available": -1,
    "max": -1
  },
  "ram_usage": 4414,
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "customp1",
      "parent": "active",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "customp3",
      "parent": "active",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS58wmANoBtT7RdPgMRCGDb37tcCQswfwVpj6NzC55D247tTMU9D",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    }
  ],
  "total_resources": null,
  "self_delegated_bandwidth": null,
  "refund_request": null,
  "voter_info": null,
  "rex_info": null
}
```

## 마무리

이 단원에서 사용자 지정 권한의 링크를 만들고 연결하고 삭제하는 방법을 학습하였습니다. 또한 사용자 지정 권한을 사용하여 계정 권한이 수행할 수 있는 액션을 제어하는 방법도 배웠습니다.
