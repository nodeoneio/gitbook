# ABI 파일의 이해

## ABI 란 무엇이며 어떻게 동작하는가.

본 챕터에서는 ABI 파일이 무엇인지 설명하며, ABI 파일이 eosio.token 컨트랙트와 어떤 관계가 있는지 알아보겠습니다.

## 개요

eosio.cdt 에서 제공하는 eosio-cpp 유틸리티를 사용하면 ABI 파일을 생성할 수 있습니다. 그러나 경우에 따라 ABI 생성시 오작동하거나 아예 동작하지 않는 상황이 발생할 수도 있습니다. 예를 들어 고급 C++ 패턴을 사용할 때 문제를 일으킬 수 있으며, 때때로 커스텀 타입들이 ABI 생성시 문제를 일으킬 수도 있습니다. 따라서 ABI 파일의 작동 방식을 이해하는 것이 중요합니다. 그 다음 필요에 따라 코드를 디버그하고 수정할 수 있습니다.

### ABI 란 무엇인가?

ABI(Application Binary Interface)는 JSON과 바이너리 표현 사이에서 사용자 액션을 변환하는 방법을 설명하는 JSON 기반 명세서 입니다. 또한 ABI 를 사용하여 데이터베이스 상태에서 JSON 으로, 혹은 JSON 에서 데이터베이스 상태로 변환하는 방법을 설명할 수 있습니다. ABI를 이용하여 컨트랙트의 명세를 기술하면 개발자와 사용자가 JSON을 통해 원활하게 컨트랙트와 상호 작용할 수 있습니다.

{% hint style="info" %}
Mandel 의 차세대 CDT 인 ANTLER 에서는 JSON ABI 대신 Google Protocol Buffers 를 사용할 계획이라는 발표가 있었습니다.
{% endhint %}

{% hint style="info" %}
컨트랙트에 전달된 메시지와 액션은 반드시 ABI를 준수하지 않아도 됩니다. 때문에 트랜잭션을 실행할 때 ABI 를 우회 할 수도 있습니다. ABI는 단순히 안내서이지 메시지를 검사하는 역할을 하는 것은 아닙니다.
{% endhint %}

### ABI 파일 만들기

먼저 내용이 없는 ABI로 부터 시작하겠습니다. 이름은 eosio.token.abi 라고 합니다.

```jsx
{
   "version": "eosio::abi/1.0",
   "types": [],
   "structs": [],
   "actions": [],
   "tables": [],
   "ricardian_clauses": [],
   "abi_extensions": [],
   "___comment" : ""
}
```

각 항목에 대한 설명은 다음과 같습니다.

### 타입(Types)

ABI는 해석(interpret)를 위한 클라이언트 또는 인터페이스를 활성화하며 심지어 컨트랙트를 위한 GUI도 생성할 수 있습니다. 이것이 일관성 있게 작동하려면, ABI 에 작성할 퍼블릭 액션이나 구조체에서 매개변수로 사용할 사용자 정의 타입을 설명해야 합니다.

{% hint style="info" %}
EOSIO는 다양한 커스텀 빌트인(built-in)을 구현해 놓았습니다. 기본으로 제공되는 타입은 따로 ABI 파일에 설명할 필요는 없습니다. EOSIO의 빌트인 기능을 깊이 알고싶다면 [여기](https://github.com/EOSIO/eos/blob/de78b49b5765c88f4e005046d1489c3905985b94/libraries/chain/abi\_serializer.cpp#L89-L127)에 정의되어 있는 내용을 확인합니다
{% endhint %}

```jsx
{
   "new_type_name": "name",
   "type": "name"
}
```

이제 전체 ABI 는 다음과 같습니다.

```jsx
{
   "version": "eosio::abi/1.1",
   "types": [{
     "new_type_name": "name",
     "type": "name"
   }],
   "structs": [],
   "actions": [],
   "tables": [],
   "ricardian_clauses": [],
   "abi_extensions": []
}
```

### 구조체(Structs)

ABI 에 노출시킬 구조체 역시 작성되어야 합니다. eosio.token.hpp 파일을 보면 퍼블릭 액션에서 어떤 구조체들이 사용되고 있는지 빠르게 파악할 수 있습니다. 특히 이는 다음 단계로 넘어가기 위한 중요한 내용입니다.

JSON 에 작성된 구조체는 다음과 같습니다.

```jsx
{
   "name": "issue", //이름
   "base": "", 			//상속해주는, 부모 구조체
   "fields": []			//구조체 필드를 설명하는 필드 오브젝트의 배열.
}
```

### 필드(Fields)

```jsx
{
   "name":"", // 필드 이름
   "type":""   // 필드 타입
}
```

eosio.token 컨트랙트에는 정의해야 할 구조체들이 많이 있습니다. 다만, 모든 구조체가 명시적으로 정의된 것은 아니며 일부는 액션의 매개 변수와 일치합니다. 다음은 eosio.token 컨트랙트에서 ABI 명세가 필요한 구조체의 목록입니다.

### 암시적 구조체

다음 구조체들은 컨트랙트에 명시적으로 정의되지 않았기 때문에 암시적 성격을 가집니다. create 액션을 보면 name 타입의 issuer 와 asset 타입의 maximum\_supply 라는 두 개의 매개 변수가 있는 것을 볼 수 있습니다. 본문에서 모든 구조체를 분석할 것은 아니지만 각각의 구조체에 동일한 논리를 적용하면 다음과 같은 결과를 얻을 수 있습니다.

#### Create

```jsx
{
  "name": "create",
  "base": "",
  "fields": [
    {
      "name":"issuer",
      "type":"name"
    },
    {
      "name":"maximum_supply",
      "type":"asset"
    }
  ]
}
```

#### issue

```jsx
{
  "name": "issue",
  "base": "",
  "fields": [
    {
      "name":"to",
      "type":"name"
    },
    {
      "name":"quantity",
      "type":"asset"
    },
    {
      "name":"memo",
      "type":"string"
    }
  ]
}
```

#### retire

```jsx
{
  "name": "retire",
  "base": "",
  "fields": [
    {
      "name":"quantity",
      "type":"asset"
    },
    {
      "name":"memo",
      "type":"string"
    }
  ]
}
```

#### transfer

```jsx
{
  "name": "transfer",
  "base": "",
  "fields": [
    {
      "name":"from",
      "type":"name"
    },
    {
      "name":"to",
      "type":"name"
    },
    {
      "name":"quantity",
      "type":"asset"
    },
    {
      "name":"memo",
      "type":"string"
    }
  ]
}
```

#### close

```jsx
{
  "name": "close",
  "base": "",
  "fields": [
    {
      "name":"owner",
      "type":"name"
    },
    {
      "name":"symbol",
      "type":"symbol"
    }
  ]
 }
```

### 명시적 구조체

이 구조체들은 다중 인덱스 테이블을 인스턴스화하기 위해 필요하기 때문에 명시적으로 정의되었습니다. 이 구조체들도 위에서 설명한 것처럼 암시적 구조체를 정의하는 것과 다르지 않습니다.

#### account

```jsx
{
  "name": "account",
  "base": "",
  "fields": [
    {
      "name":"balance",
      "type":"asset"
    }
  ]
}
```

### 액션(Actions)

액션 JSON 오브젝트의 정의는 다음과 같습니다.

```jsx
{
  "name": "transfer", 		// 컨트랙트에 정의된 액션의 이름
  "type": "transfer", 		// ABI에 기술된 암시적 구조체의 이름
  "ricardian_contract": "" 	// (옵션)이 액션에 대한 리카르디안 구절. 의도한 기능을 설명.
}
```

eosio.token 컨트랙트의 헤더 파일에 설명된 모든 퍼블릭 함수를 모아서 eosio.token 계약의 액션을 설명합니다.

그 다음 앞에서 설명한 구조체와 관련된 각 액션의 타입을 설명합니다. 대부분의 경우 함수 이름과 구조체 이름이 동일하지만 그렇다고 반드시 같을 필요는 없습니다.

다음 예제는 JSON 과 소스코드를 연결하는 액션의 목록입니다. 액션이 어떻게 기술되는지 볼 수 있습니다.

#### create

```jsx
{
  "name": "create",
  "type": "create",
  "ricardian_contract": ""
}
```

#### issue

```jsx
{
  "name": "issue",
  "type": "issue",
  "ricardian_contract": ""
}
```

#### retire

```jsx
{
  "name": "retire",
  "type": "retire",
  "ricardian_contract": ""
}
```

#### transfer

```jsx
{
  "name": "transfer",
  "type": "transfer",
  "ricardian_contract": ""
}
```

#### close

```jsx
{
  "name": "close",
  "type": "close",
  "ricardian_contract": ""
}
```

### 테이블(Tables)

다음은 테이블의 JSON 오브젝트 정의 입니이다.

```jsx
{
  "name": "",       // 인스턴스화 할 때 결정되는 테이블의 이름.
  "type": "",       // 이 테이블과 관련된 구조체의 이름
  "index_type": "", // 이 테이블의 프라이머리 인덱스 타입.
  "key_names" : [], // 키 이름의 배열. 길이는 key_types 멤버의 길이와 같이야 한다.
  "key_types" : []  // 키 이름 배열 멤버와 관련된 키 타입들의 배열. 키 이름 멤버 배열의 길이와 같아야 한다.
}
```

eosio.token 컨트랙트는 accounts 와 stat 두 개의 테이블을 인스턴스화 합니다.

accounts 테이블은 account 구조체를 기반으로 하며 uint64를 Primary Key로 가지는 i64 인덱스 타입 입니다.

이 테이블을 ABI로 기술하는 방법은 아래와 같습니다.

```jsx
{
  "name": "accounts",
  "type": "account", // 이전에 정의된 구조체 이름과 일치함.
  "index_type": "i64",
  "key_names" : ["primary_key"],
  "key_types" : ["uint64"]
}
```

stats 테이블은 currency\_stats 구조체를 기반으로 하며 uint64를 Primary Key로 가지는 i64 인덱스 타입 입니다.

이 테이블을 ABI로 기술하는 방법은 아래와 같습니다.

```jsx
{
  "name": "stats",
  "type": "currency_stats",
  "index_type": "i64",
  "key_names" : ["primary_key"],
  "key_types" : ["uint64"]
}
```

위 두 개의 테이블 들은 "primary\_key" 라는 동일한 key name 이 있는 것을 볼 수 있습니다. 키의 이름을 기능과 비슷한 이름으로 지정하면 본질적으로 어떠한 관계 있는지를 나타낼 수 있습니다. 이 구현과 마찬가지로 주어진 값은 다른 테이블을 쿼리하는 데 사용 할 수 있음을 의미합니다.

### 전체 ABI

eosio.token 전체를 기술하는 ABI 파일은 다음과 같습니다.

```jsx
{
  "version": "eosio::abi/1.1",
  "types": [
    {
      "new_type_name": "name",
      "type": "name"
    }
  ],
  "structs": [
    {
      "name": "create",
      "base": "",
      "fields": [
        {
          "name":"issuer",
          "type":"name"
        },
        {
          "name":"maximum_supply",
          "type":"asset"
        }
      ]
    },
    {
       "name": "issue",
       "base": "",
       "fields": [
          {
            "name":"to",
            "type":"name"
          },
          {
            "name":"quantity",
            "type":"asset"
          },
          {
            "name":"memo",
            "type":"string"
          }
       ]
    },
    {
       "name": "retire",
       "base": "",
       "fields": [
          {
            "name":"quantity",
            "type":"asset"
          },
          {
            "name":"memo",
            "type":"string"
          }
       ]
    },
    {
       "name": "close",
       "base": "",
       "fields": [
          {
            "name":"owner",
            "type":"name"
          },
          {
            "name":"symbol",
            "type":"symbol"
          }
       ]
    },
    {
      "name": "transfer",
      "base": "",
      "fields": [
        {
          "name":"from",
          "type":"name"
        },
        {
          "name":"to",
          "type":"name"
        },
        {
          "name":"quantity",
          "type":"asset"
        },
        {
          "name":"memo",
          "type":"string"
        }
      ]
    },
    {
      "name": "account",
      "base": "",
      "fields": [
        {
          "name":"balance",
          "type":"asset"
        }
      ]
    },
    {
      "name": "currency_stats",
      "base": "",
      "fields": [
        {
          "name":"supply",
          "type":"asset"
        },
        {
          "name":"max_supply",
          "type":"asset"
        },
        {
          "name":"issuer",
          "type":"name"
        }
      ]
    }
  ],
  "actions": [
    {
      "name": "transfer",
      "type": "transfer",
      "ricardian_contract": ""
    },
    {
      "name": "issue",
      "type": "issue",
      "ricardian_contract": ""
    },
    {
      "name": "retire",
      "type": "retire",
      "ricardian_contract": ""
    },
    {
      "name": "create",
      "type": "create",
      "ricardian_contract": ""
    },
    {
      "name": "close",
      "type": "close",
      "ricardian_contract": ""
    }
  ],
  "tables": [
    {
      "name": "accounts",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["currency"],
      "key_types" : ["uint64"]
    },
    {
      "name": "stats",
      "type": "currency_stats",
      "index_type": "i64",
      "key_names" : ["currency"],
      "key_types" : ["uint64"]
    }
  ],
  "ricardian_clauses": [],
  "abi_extensions": []
}
```

### 토큰 컨트랙트에서 다룰 수 없는 경우

#### 벡터(vector)

ABI 파일에 벡터를 기술할 때는 타입에 \[]를 추가하기만 하면 됩니다. 예를 들어 권한 수준을 벡터로 나타내야 한다면 permission\_level\[] 와 같이 작성 할 수 있습니다.

#### 구조체 베이스(Struct Base)

거의 사용되지 않기 때문에 딱히 언급할 의미가 없는 속성입니다. 상속을 하려면 base ABI structure 속성을 사용하여 다른 구조체를 참조할 수 있는데, 그 구조체는 동일한 ABI 파일에 기술되어 있어야 합니다. 스마트 컨트랙트 로직이 상속을 지원하지 않는 경우라면 베이스는 아무런 동작도 하지 않거나 오히려 오류를 발생시킬 수 있습니다.

[시스템 컨트랙트 소스 코드](https://github.com/EOSIO/eosio.contracts/blob/4e4a3ca86d5d3482dfac85182e69f33c49e62fa9/eosio.system/include/eosio.system/eosio.system.hpp#L46) 및 [ABI](https://github.com/EOSIO/eosio.contracts/blob/4e4a3ca86d5d3482dfac85182e69f33c49e62fa9/eosio.system/abi/eosio.system.abi#L262)에서 사용 중인 base의 예를 볼 수 있다.

#### 추가 ABI 속성

간결성을 위해 ABI 규격의 몇 가지 속성을 생략했지만, 현재 보류 중인 ABI 규격 중에는 ABI의 모든 속성에 대한 개요를 제공하는 경우도 있습니다.

#### 리카르디안 구절(Ricardian Clauses)

리카르디안 구절은 특정 액션이 의도하는 바를 설명합니다. 또한 발신자와 컨트랙트 사이의 조건을 설정하는 데 사용될 수도 있습니다.

#### 확장 ABI

구 버전 클라이언트에서 확장 데이터의 "chunks" 구문 분석을 건너뛸 수 있는 일반 "미래 증명 future proofing" 계층입니다. 현재 이 속성은 사용되지 않습니는다. 앞으로는 각 확장자가 해당 벡터에 고유한 "chunks"를 가지게 되므로, 구 버전 클라이언트는 건너뛰고 이를 해석하는 방법을 이해하는 새로운 클라이언트를 사용하는 것이 좋습니다.

### 유지 관리

구조체 변경, 테이블 추가, 액션 추가 또는 매개 변수 추가, 새로운 타입 도입 등과 같은 상황이 발생하면 ABI 파일을 업데이트해야 합니다. 하지만 대부분의 경우, ABI 파일 업데이트를 잊어버린다고 오류가 발생하지는 않습니다.

### 문제 해결

#### 테이블이 행을 반환하지 않음

테이블이 ABI 파일에 정확하게 기술되어 있는지 확인합니다. 예를 들어 cleos를 사용하여, 잘못된 형식의 ABI 정의가 있는 컨트랙트에 테이블을 추가한 다음 해당 테이블에서 행을 가져오면 아무 결과도 표시되지 않습니다. cleos는 행을 추가하거나 읽을 때 컨트랙트가 ABI 내에 테이블을 잘못 기술하고 있는 속성을 가지고 있더라도 오류를 반환하지는 않습니다.
